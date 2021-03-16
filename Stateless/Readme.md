Scope:
    Create stateless  Redis application in the namespace called "stateless"
    in the existing EKS cluster
Exit creteria:
    Running redis Guestbook at frontend
    with master / slave backend in "stateless" workspace.


Solution:

First step is creation of the new workspace "stateless":
    kubectl create namespace stateless

Then I made separate yaml manifest files for deployments: redis-master, redis-slave, redis-frontend
and according service definitions for this deployments: redis-service, redis-slave-service, redis-frontend-service
Then all of these manifest files were applied with kubectl:
    kubectl apply -f <filename>

We had working Guestbook application in result (see guestbook.png)
