# kubernetes
Kubernetes playground

# From dashboard directory.

1. Run the influxdb.yaml

              kubectl apply -f influxdb.yaml             
2. Run the heapster.yaml

       kubectl apply -f heapster.yaml
3. Run the dashboard.yaml
       
        kubectl apply -f dashboard.yaml
4. Run the sa_cluster_admin.yaml

        kubectl apply -f sa_cluster_admin.yaml

5. To get the login Token use the below commands.

            kubectl describe sa dashboard-admin -n kube-system  #This will give the secret used for the service account.
     
6. List the secret use for the dashbaord-admin service account and then run the describe cmd on the secret  to get the token value.

              kubectl describe secret dashboard-admin-token-vg234  # output the token and then use that token to login to the dashboard
     


# Steps to enable user-name password based authentication.
Below changes are to be performed after following the guide from the above dashboard installation process from the dashboard directory.
1. In the dashboard.yml deploymnent section pass an extra arg 
        
              args:
                - --auto-generate-certificates
                - --authentication-mode=basic   #add this value in dashboard.yml
2. This will enable Basic login in the dashboard login page.

3. Now in the api server config file available at "/etc/kubernetes/manifests/kube-apiserver.yaml"  add the below option.

       spec:
        containers:
        - command:
          - kube-apiserver
          
          - --basic-auth-file=/etc/kubernetes/pki/user.csv  # Add this Line under the conatiner command section. The auth file location can be any in the server
          
4. Also create a password file of formate .csv which containe below values.
                
                  password,user,uid    # Content of user.csv file

5. Basically after updating the api-server file the api-service pod will restart by it self if not then restart the pod.

5. After making above changes we need to map the user to a role it can be a admin or a viwer role, for which update the "sa_cluster_admin.yaml" service account file with the below role for the user.

            apiVersion: v1
            kind: ServiceAccount
            metadata:
              name: dashboard-admin
              namespace: kube-system
            ---
            apiVersion: rbac.authorization.k8s.io/v1beta1
            kind: ClusterRoleBinding
            metadata:
              name: cluster-admin-rolebinding
            roleRef:
              apiGroup: rbac.authorization.k8s.io
              kind: ClusterRole
              name: cluster-admin
            subjects:
            - kind: ServiceAccount
              name: dashboard-admin
              namespace: kube-system
            - kind: User                      #add this to the service account section
              name: admin                     #user name which you have created
              namespace: kube-system
