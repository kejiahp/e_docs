# AWS ECS Application Load-Balancer Setup Guide

1. Target group (the ALB's backend)
   - Type: IP addresses (required for Fargate/awsvpc — not "Instances")
   - Protocol/port: HTTP : 8000
   - VPC: same default VPC as your service
     Health check path: /api/v1 (your base route returns 200) — or /health if you add one; success codes 200

2. Security groups (two of them)
   - ALB SG: inbound 80 (and 443 later) from 0.0.0.0/0.
   - Task SG: inbound 8000 from the ALB SG — and remove the 8000 from 0.0.0.0/0 rule you added earlier. Traffic should reach tasks only through the ALB, not directly.

3. Create the ALB
   - Internet-facing, IPv4, in 2+ public subnets (default VPC subnets are public).
   - Attach the ALB SG.
   - Listener: HTTP :80 → forward to the target group from step 1.

4. Attach the ALB to your ECS service
   - Update the service's load-balancer config to register to that target group on container <container-name>:8000. (Adding an LB to an existing service is supported via update-service for rolling deployments, but if it errors you may need to recreate the service with the LB attached.)
   - Set healthCheckGracePeriodSeconds to ~60–120s so ECS doesn't kill tasks before the app boots (this also fixes the "stuck on wait-for-stability" you saw — the service now stabilizes once the target is healthy).
     The service will auto-register/deregister each task's IP in the target group on every deploy.

5. Grab the DNS name → give to the frontend
   The ALB's DNS name is your endpoint:

http://<alb-name>.eu-west-2.elb.amazonaws.com/api/v1
curl that — you should get the base Message 200.

The free `\*.elb.amazonaws.com` name only serves HTTP, and you can't get an ACM certificate for it (ACM needs a domain you control). If your frontend is served over HTTPS, browsers will block HTTP API calls (mixed content).

Options:

- Free HTTPS: put a CloudFront distribution in front of the ALB → you get a free \*.cloudfront.net HTTPS URL with an AWS-managed cert. (Disable caching for the API behavior.) This is the closest to "free public HTTPS DNS from AWS."

- Custom domain: register a domain (Route 53, ~$12/yr — not free), get a free ACM cert, add an HTTPS :443 listener to the ALB, and a Route 53 alias record to the ALB. Cleanest long-term.
  If you only need HTTP for now (dev), the ALB name works as-is.
  A couple of things I can do to help:

### Setting Up Cloudfront distribution for HTTPS

**Trick**: CloudFront is HTTPS (free \*.cloudfront.net cert), CloudFront points to the ALB stays HTTP on :80, so you don't need a cert on the ALB at all.

1. ALB: keep its HTTP :80 listener (that's the origin).

2. Create the CloudFront distribution:

Origin domain: your ALB DNS name (…elb.amazonaws.com)
Origin protocol: HTTP only, port 80

3. Default cache behavior (this is the part people get wrong for APIs):

   > Viewer protocol policy: Redirect HTTP → HTTPS
   >
   > Allowed methods: GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE (default is GET/HEAD only — your API needs the rest)
   >
   > Cache policy: CachingDisabled (managed)
   >
   > Origin request policy: AllViewerExceptHostHeader (managed) — forwards everything but lets the ALB see its own host

4. Create, wait ~a few minutes for "Deployed", then your API base is:

https://dXXXXXXXX.cloudfront.net/api/v1 Give that to your frontend.

**One gotcha**: don't forward the Host header to the ALB (that's why AllViewerExceptHostHeader) — forwarding CloudFront's host can break ALB routing/health.
