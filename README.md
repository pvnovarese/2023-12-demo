# 2023-12-demo

Simple demo for Anchore Enterprise.

Includes workflow examples for Jenkins, CircleCI, Codefresh, Drone, and GitHub.

Partial list of conditions that can be tested with this image:

1. xmrig cryptominer installed at `/xmrig/xmrig`
2. simulated AWS access key in `/aws_access`
3. simulated ssh private key in `/ssh_key`
4. selection of commonly-blocked packages installed (sudo, curl, etc)
5. `/log4j-core-2.14.1.jar` (CVE-2021-44228, et al)
6. added anchorectl to demonstrate automatic go library detection in binaries
7. wide variety of ruby, node, python, java installed with different licenses
8. build drift detection via baseline dockerfile with minimal packages/dependencies
9. Terraform RPM with BUSL license installed

To do:
1. add secret scanning to distributed scan
