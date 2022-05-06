## Set Up htpasswd as the Identity Provider and Add Users and Permissions

1. Create htpasswd file with these users with the password doubletap:
- columbus
- wichita
- littlerock
- tallahassee
- admin
```diff
+ htpasswd

```
2. Create HTPasswd Secret from file.
3. Download the HTPasswd Custom Resource (using the link provided on the lab page).
4. Add the name of your HTPasswd Secret to the file.
5. Apply your custom resource to your cluster.
6. Create a project called zLand.
7. Give columbus admin permissions to the zLand project.
8. Give wichita and littlerock edit permissions to the zLand project.
9. Give tallahassee basic user permissions to the zLand project.
10. Give admin cluster admin permissions.
11. Remove the kubeadmin user from the cluster.

## Role-Based Access and Groups
1. Create a project called twinkies.
2. Create a group called yum.
3. Add columbus, wichita, and littlerock to the yum group.
4. Grant admin access to the yum group for the project twinkies.
5. Create a custom resource that allows tallahassee to get pod information from the twinkies project and call it gettwinkies.

## Quotas and Resource Limits
1. Download the quota and resource limit templates (using the links provided on the lab page).
2. Modify the quota.yaml file with the following values:
   - Max number of pods = 3
   - Max amount of memory = 2 GB
   - Max number of replication controllers = 2
   - Max number of services allowed = 8
3. Create quota and apply it to the zLand project.
4. Modify the resource_limits.yaml file with the following values:
   - Max number of pods = 4
   - Requested cpus = 1
   - Requested memory = 1 GB
   - Requested Ephemeral storage = 2 GB
   - Limit cpus to 4
   - Limit memory to 4 GB
   - Limit Ephemeral storage to 8 GB
5. Create resource limit and apply it to the twinkies project.

## Application Creation and Management
Note: Use https://github.com/sclorg/cakephp-ex example app to create applications

1. Create test-app1, test-app2, test-app3, test-app4, and test-app5 projects.
2. Create an application named cake and make sure it is accessible to the outside world in project test-app1.
3. Create an application using a route called twinkiesforall in the test-app2 project.
4. Create an application using a secured route called mytwinkie in the test-app3 project. Use self-signed cert from lab repo to secure the route (the links are provided on the lab page).
5. Create an application that can use the dont-tell secret project test-app4.
6. Create a secret called dont-tell in the test-app4 project. Download the secret.yaml file (using the link provided on the lab page).
7. Populate with the user dXN1ci1uYW11 and password dGHcz4dvCmQ=.
8. Create a service account called madison in the test-app5 project.
9. Create an application that can be edited by the madison service account in the test-app5 project.
10. Manually scale the application in the test-app2 project to 2 pods.
11. Set an autoscaler for min of 1 pod and a max of 3 pods based of 75% CPU utilization for the application in the test-app5 project.
