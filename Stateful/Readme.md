Useful command:
    kubectl patch storageclass my-class -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}' --namespace=<your namespace>