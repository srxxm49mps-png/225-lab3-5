# 225-lab3-5
CI/CD pipeline Example with Linting on DEV deployment, and to PROD if no linting errors.  Will likely fail on the first run.  Rename robots1.txt to robots.txt to remove the linting error.

Lints the HTML before deploying to a DEV environment using the __ClusterIP with Ingress__. Changes the Dockerfile to Dockerfile.build in preparation for future steps. Then deploys to a PROD environment at the __LoadBalanced IP__.

1) Create a new blank repository in your GitHub account using the lab name above.
2) Copy the URL of this repository and paste it into your repository.
3) View each file, and make changes where it is commented.
4) Run your pipeline.
5) Demonstrate your changes to the code in your video, and show the resulting web page.

## This deployment

DEV: http://10.48.229.166/lab3-5/

PROD: http://10.48.229.166:8081/

Jenkins builds with `-f Dockerfile.build`. The current course template already
names its copied file `robots.txt`.
Production uses the existing MetalLB v0.14.3 installation and the user's
single-IP pool. Lab 3.4 uses port 8080; this lab uses port 8081. Both services
opt into IP sharing with the same annotation while retaining separate selectors.
The pipeline waits for production rollout and external IP assignment, then
checks the Lab 3.5 page before reporting success.
