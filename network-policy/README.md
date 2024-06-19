![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/55130110-90d8-41a7-829b-e8fa63788c12)


**Step 1: Label Your Namespaces**

First, label your namespaces to make them identifiable in your network policies.

**Backend Namespace:**

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/6af21263-d317-4e95-a4df-5a2008947b03)




**Database Namespace:**

![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/ba0762e4-e7d6-4570-b228-99d2abc5db40)



kubectl apply -f backend-namespace.yaml
kubectl apply -f database-namespace.yaml


**Step 2: Create the Network Policy in the Database Namespace**

Next, create a Network Policy that allows ingress traffic to the Database namespace only from the Backend namespace.

**Network Policy YAML:**


![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/b9bb8a14-aaf3-49fe-b97a-f73ac410c14e)



kubectl apply -f allow-backend-to-database.yaml


![image](https://github.com/Shubham2194/kubernetes-CKA/assets/83746560/b3662fa2-783b-438f-82aa-af47a9680ba2)



**COOL!**
