In today's AI-driven world, where terms like RAG (Retrieval-Augmented Generation) and GenAI (Generative AI) are becoming increasingly popular, I've realized how critical it is to manage data effectively. The rise of vector databases has shown that the right database choice isn't just about storage, it's about empowering AI applications to deliver real-time insights and advanced capabilities. From my experience, picking the right database can make or break an AI project, especially when dealing with the complex data needs of modern AI solutions.

Databases are foundational components of computing environments, enabling efficient data storage and management. The choice of a database often depends on an application's specific needs, such as real-time analytics, horizontal scaling requirements, or the complexity of transactions it must support. In this blog, I will discuss one such database, [SingleStore](https://msql.co/deploying-genai), and demonstrate how to deploy it in a cloud-native environment.

## Prerequisites

- Basic understanding of [Docker](https://www.docker.com/) and [Kubernetes](https://kubernetes.io/).
- Knowledge of [YAML](https://yaml.org/) configurations.
- An account on [Civo](https://www.civo.com/) and a basic understanding of cloud computing.

## Challenges Faced with Traditional Databases

**Traditional databases** present several challenges that can affect performance and scalability in modern applications. Below are some of the most common issues:

1. **Long Query Execution Times**  
   Traditional databases can struggle with long execution times for complex queries, particularly when dealing with large volumes of data. Businesses may lose millions of dollars due to delays in real-time data analysis, which can lead to poor decision-making.
2. **Data Storage Limitations**  
   Many traditional database systems have limitations on how much data can be effectively stored and managed. These limitations also impact the performance of applications, especially when scaling to handle large volumes of data.
3. **Limited Deployment Flexibility**  
   Traditional databases may not be easily deployable in cloud-native environments, posing a significant challenge for companies looking to adopt new technologies.
4. **High Cost of Scaling and Maintenance**  
   Scaling traditional databases often requires additional physical servers or instances, which can be expensive and complex to manage.

## How SingleStore Solves Common Database Challenges

[SingleStore](https://msql.co/deploying-genai) is a versatile database designed to handle complex SQL, JSON, and vector workloads. It helps developers overcome the challenges of traditional databases with innovative features and capabilities:

1. **Accelerated Query Performance**  
   SingleStore reduces query execution times from hours to mere seconds or minutes, making it a game-changer in the world of Generative AI, where quick data retrieval is essential.
2. **Expanded Storage Capabilities**  
   Are you stuck with a system that can only store data for 90 days? With such **limited capacity**, analyzing long-term trends or making data-driven predictions becomes nearly impossible. SingleStore changes the game by offering **scalable storage** that allows you to retain and analyze data for much longer periods—even decades.
3. **Flexible Deployment Options**  
   With **SingleStore**, you can deploy your application wherever it suits your use case best, whether on-premises, in the cloud, or in containers using **Kubernetes and Docker**.
4. **Cost Efficiency: High Performance, Lower Costs**  
   SingleStore offers a much more cost-effective solution, delivering top-tier performance at a lower cost.

## Key Features of SingleStore

SingleStore comes with various features that help overcome the limitations of traditional databases. Below are some of the key features:

1. **Real-Time Analysis**  
   SingleStore is used for real-time analytics, enabling businesses to execute complex queries on live data with minimal latency. This is essential for industries like financial services, e-commerce, and IoT, where real-time analysis is crucial.
2. **Scalability**  
   SingleStore scales horizontally by adding more nodes to the cluster, allowing you to scale your database according to your needs.
3. **ACID Compliance**  
   SingleStore supports **ACID properties**, ensuring that your data remains reliable even in the most critical environments.
4. **Hybrid Transactional/Analytical Processing (HTAP)**  
   SingleStore bridges the gap between transactional and analytical processing due to its HTAP feature, allowing both to exist in a single system. This reduces the need for multiple databases, simplifying the overall architecture.

## Leverage SingleStore Kubernetes Operator

In this section, I will demonstrate how to set up SingleStore in your Kubernetes cluster and run MySQL.

### Step 1: Create a Kubernetes Cluster on Civo

![image](https://hackmd.io/_uploads/B1Ifvh_9A.png)

I will be using [Civo's](https://www.civo.com/) Kubernetes offering to set up a Kubernetes cluster. You can visit [Civo Documentation](https://www.civo.com/docs/kubernetes/create-a-cluster) to create a Kubernetes cloud cluster. We are going to create a 4-node medium-performance cluster.

![Screenshot 2024-08-13 at 11.37.42 AM (2)](https://hackmd.io/_uploads/HkiYP2O50.png)

Once the cluster is up and running, you can download the kubeconfig file and export it to your local machine to use the Kubernetes cloud cluster created on Civo or use the [Civo CLI](https://www.civo.com/docs/overview/civo-cli).

To verify the setup, you can run the following command, which will list the number of nodes in your Kubernetes cluster:

```
kubectl get nodes

NAME                                                    STATUS   ROLES    AGE    VERSION
k3s-singlestore-demo-ece0-845f22-node-pool-7077-c3aod   Ready    <none>   4h53m  v1.28.7+k3s1
k3s-singlestore-demo-ece0-845f22-node-pool-7077-lx7zr   Ready    <none>   4h53m  v1.28.7+k3s1
k3s-singlestore-demo-ece0-845f22-node-pool-7077-fixzt   Ready    <none>   4h53m  v1.28.7+k3s1
k3s-singlestore-demo-ece0-845f22-node-pool-7077-ifcsi   Ready    <none>   4h53m  v1.28.7+k3s1
```

To list the number of pods running, you can use the following command:

```
kubectl get pods

NAME                                  READY   STATUS     RESTARTS   AGE
install-traefik2-nodeport-si-qnpgr    0/1     Completed  0          4h58m

```

**NOTE: SingleStore Kubernetes Operator requires a cluster with 4 nodes. Each node should have 4 CPU cores and a minimum of 16GB RAM.** 

### Step 2: Pull SingleStore Docker Image

Once the cluster setup is complete, you can pull the [SingleStore Operator](https://hub.docker.com/r/singlestore/operator) from Docker Hub using the following command:

```
docker pull singlestore/operator:3.258.0-f5ba0d6a

3.258.0-f5ba0d6a: Pulling from singlestore/operator
c129dd76092a: Pull complete
23034e1e5ba6: Pull complete
6718c92c55bf: Pull complete
Digest: sha256:a733109632b968b8ef9b77e4f805088a631ed9cfba26dec0b84815f10404a386
Status: Downloaded newer image for singlestore/operator:3.258.0-f5ba0d6a
docker.io/singlestore/operator:3.258.0-f5ba0d6a
```

**NOTE: Before performing the above step, ensure Docker is up and running.**

### Step 3: Create the Object Definition Files

First, store the following values in a **sdb-rbac.yaml** file, which will create a **Role-Based Access Control (RBAC)** manifest that generates a ServiceAccount, Role, and RoleBinding object for use with the Operator.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sdb-operator
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: sdb-operator
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - services
  - endpoints
  - persistentvolumeclaims
  - events
  - configmaps
  - secrets
  verbs:
  - '*'
- apiGroups:
  - policy
  resources:
  - poddisruptionbudgets
  verbs:
  - '*'
- apiGroups:
  - batch
  resources:
  - cronjobs
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - namespaces
  verbs:
  - get
- apiGroups:
  - apps
  - extensions
  resources:
  - deployments
  - daemonsets
  - replicasets
  - statefulsets
  - statefulsets/status
  verbs:
  - '*'
- apiGroups:
  - memsql.com
  resources:
  - '*'
  verbs:
  - '*'
- apiGroups:
  - networking.k8s.io
  resources:
  - networkpolicies
  verbs:
  - '*'
- apiGroups:
  - ""
  resources:
  - serviceaccounts
  verbs:
  - get
  - watch
  - list
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: sdb-operator
subjects:
- kind: ServiceAccount
  name: sdb-operator
roleRef:
  kind: Role
  name: sdb-operator
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1


kind: ClusterRole
metadata:
  name: sdb-operator
rules:
- apiGroups:
  - storage.k8s.io
  resources:
  - storageclasses
  verbs:
  - get
  - list
  - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: sdb-operator
subjects:
- kind: ServiceAccount
  name: sdb-operator
  namespace: default
roleRef:
  kind: ClusterRole
  name: sdb-operator
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: backup
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: backup
rules:
- apiGroups: ["batch"]
  resources: ["jobs"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: backup
subjects:
- kind: ServiceAccount
  name: backup
roleRef:
  kind: Role
  name: backup
  apiGroup: rbac.authorization.k8s.io
```

Create the resource using the following command:

```
kubectl create -f sdb-rbac.yaml

serviceaccount/sdb-operator created
role.rbac.authorization.k8s.io/sdb-operator created
rolebinding.rbac.authorization.k8s.io/sdb-operator created
clusterrole.rbac.authorization.k8s.io/sdb-operator created
clusterrolebinding.rbac.authorization.k8s.io/sdb-operator created
serviceaccount/backup created
role.rbac.authorization.k8s.io/backup created
rolebinding.rbac.authorization.k8s.io/backup created
```

After creating the RBAC, create a Custom Resource Definition (CRD) for the MemsqlCluster resource by saving the following code in a **sdb-cluster-crd.yaml** file.

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: memsqlclusters.memsql.com
spec:
  group: memsql.com
  names:
    kind: MemsqlCluster
    listKind: MemsqlClusterList
    plural: memsqlclusters
    singular: memsqlcluster
    shortNames:
      - singlestore
      - singlestoredb
      - memsql
  scope: Namespaced
  versions:
  - name: v1alpha1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        description: Schema for the SingleStore Cluster
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation
              of an object. Servers should convert recognized schemas to the latest
              internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this
              object represents. Servers may infer this from the endpoint the client
              submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: Spec defines the desired state of Cluster
            type: object
            x-kubernetes-preserve-unknown-fields: true
          status:
            description: Status defines the observed state of Cluster
            type: object
            x-kubernetes-preserve-unknown-fields: true
        type: object
    subresources:
      status: {}
    additionalPrinterColumns:
    - name: Aggregators
      type: integer
      description: Number of Aggregators
      jsonPath: .status.expectedAggregators
    - name: Leaves
      type: integer
      description: Number of Leaf Nodes (per availability group)
      jsonPath: .status.expectedLeaves
    - name: Redundancy Level
      type: integer
      description: Redundancy level of the Cluster
      jsonPath: .spec.redundancyLevel
    - name: Age
      type: date
      jsonPath: .metadata.creationTimestamp
```

Create the resource using the following command:

```
kubectl create -f sdb-cluster-crd.yaml
customresourcedefinition.apiextensions.k8s.io/memsqlclusters.memsql.com created
```

Once the CRD and RBAC are configured, create an Operator Deployment object that will spawn and maintain the Operator by saving the following code in a **sdb-operator.yaml** file.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sdb-operator
  labels:
    app.kubernetes.io/component: operator
spec:
  replicas: 1
  selector:
    matchLabels:
      name: sdb-operator
  template:
    metadata:
      labels:
        name: sdb-operator
    spec:
      serviceAccountName: sdb-operator
      containers:
        - name: sdb-operator
          image: <singlestore/operator docker image>
          imagePullPolicy: Always
          args: [
            # Cause the operator to merge rather than replace annotations on services
            "--merge-service-annotations",
            # Allow the process inside the container to have read/write access to the `/var/lib/memsql` volume.
            "--fs-group-id", "5555",
            "--cluster-id", "sdb-cluster"          ]
          env:
            - name: WATCH_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: OPERATOR_NAME
              value: "sdb-operator"
```

Change the value of the container image under the `spec` section to the SingleStore Operator Docker image, e.g., `image: singlestore/operator:3.258.0-f5ba0d6a`.

Create the resource using the following command:

```
kubectl create -f sdb-operator.yaml
deployment.apps/sdb-operator created
```

The above resource should create a pod with a name starting from **sdb-operator**. You can verify the pod creation using the following command:

```
kubectl get pods
```

Wait for the new pod to be created, and proceed once the pod status is running.

After creating all the above resources, create a MemsqlCluster object (from the custom resource definition) that spawns the cluster and all associated necessary objects/resources based on the configurations supplied within by saving the following code in a **sdb-cluster.yaml** file.

```yaml
apiVersion: memsql.com/v1alpha1
kind: MemsqlCluster
metadata:
  name: sdb-cluster
spec:
  license: license_key
  adminHashedPassword: "hashed_password"
  nodeImage:
    repository: singlestore/node
    tag: node_tag

  redundancyLevel: 2

  serviceSpec:
    objectMetaOverrides:
      labels:
        custom: label
      annotations:
        custom: annotations

  aggregatorSpec:
    count: 2
    height: 0.5
    storageGB: 256
    storageClass: standard

    objectMetaOverrides:
      annotations:
        optional: annotation
      labels:
        optional: label

  leafSpec:
    count: 2
    height: 0.5
    storageGB: 1024
    storageClass: standard

    objectMetaOverrides:
      annotations:
        optional: annotation
      labels:
        optional: label
```

Replace **license_key** with your license from the [Cloud Portal](https://).

Replace **hashed_password** with a hashed version of a secure password for the admin database user on the cluster.

The following Python script shows how to create a hashed password. Note that the asterisk (*) must be included in the final password.

```python
from hashlib import sha1
print("*" + sha1(sha1('secretpass'.encode('utf-8')).digest()).hexdigest().upper())
```

Create the resource using the following command:

```
kubectl create -f sdb-cluster.yaml
memsqlcluster.memsql.com/sdb-cluster created
```

The above resource should create a few pods with names starting from **node-sdb-cluster**. You can verify the pod creation using the following command:

```
kubectl get pods

NAME                                READY   STATUS      RESTARTS   AGE
install-traefik2-nodeport-si-qnpgr  0/1   Completed   0          5h3m
sdb-operator-6c5f67f5b6-2tc12       1/1   Running     0          4m6s
node-sdb-cluster-aggregator-0       0/1   Running     0          3m4s
node-sdb-cluster-leaf-ag1-1         0/1   Running     0          3m4s
node-sdb-cluster-master-0           2/2   Running     0          3m32s
node-sdb-cluster-leaf-ag1-0         1/1   Running     0          3m4s
```

### Step 4: Connect to Your Cluster

After all the pods are up

 and running, also check for the services created using the following command:

```
kubectl get services

NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)              AGE
kubernetes              ClusterIP      10.43.0.1       <none>          443/TCP              5h36m
sdb-operator            ClusterIP      10.43.222.191   <none>          9090/TCP,6060/TCP    35m
svc-sdb-cluster         ClusterIP      None            <none>          3306/TCP             35m
svc-sdb-cluster-ddl     LoadBalancer   10.43.157.50    74.220.16.228   3306:30487/TCP       35m
svc-sdb-cluster-dml     LoadBalancer   10.43.124.20    74.220.16.107   3306:30483/TCP       35m
```

The Operator creates two services for use with clients and database users. Use these IP addresses from the **CLUSTER-IP** column to access these services. If the Operator cluster resides on a cloud service provider, use the IP addresses from the **EXTERNAL-IP** column instead, which can be reached externally.

### Step 5: Connect with the MySQL Client

To connect to the SingleStore cluster, run the following command from a computer that can access the Kubernetes cluster. Use the admin database user with the password you defined in the sdb-cluster.yaml definition file.

```bash
~/singlestore-demo
mysql -u admin -h 74.220.16.228 -p

Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 346
Server version: 5.7.32 SingleStoreDB source distribution (compatible; MySQL Enterprise & MySQL Commercial)

Copyright (c) 2000, 2024, Oracle and/or its affiliates.
Oracle is a registered trademark of Oracle Corporation and/or its affiliates. Other names may be trademarks of their respective owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

If you used the above Python code to generate the hashed password, you can log in to the MySQL Client by entering **secretpass** as the password.

### Step 6: Run MySQL Queries

You can now run queries and perform various database tasks. Visit [SingleStore Docs](https://docs.singlestore.com/db/v8.7/query-data/basic-query-examples/) to explore further steps.

I have attached outputs of a few queries involving the creation of tables and storing data in them.

```sql
mysql> CREATE DATABASE memsql_example;
Query OK, 1 row affected (3.21 sec)

mysql> use memsql_example;
Database changed

mysql> CREATE TABLE departments (
    -> id int,
    -> name varchar(255),
    -> PRIMARY KEY (id)
    -> );
Query OK, 0 rows affected (0.33 sec)

mysql> CREATE TABLE employees (
    -> id int,
    -> deptId int,
    -> managerId int,
    -> name varchar(255),
    -> hireDate date,
    -> state char(2),
    -> PRIMARY KEY (id)
    -> );
Query OK, 0 rows affected (0.23 sec)

mysql> CREATE TABLE salaries (
    -> employeeId int,
    -> salary int,
    -> PRIMARY KEY (employeeId)
    -> );
Query OK, 0 rows affected (0.22 sec)

mysql> INSERT INTO departments (id, name) VALUES
    -> (1, 'Marketing'), (2, 'Finance'), (3, 'Sales'), (4, 'Customer Service');
Query OK, 4 rows affected (0.24 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> INSERT INTO employees (id, deptId, managerId, name, hireDate, state) VALUES
    -> (1, 2, NULL, "Karly Steele", "2011-08-25", "NY"),
    -> (2, 1, 1, "Rhona Nichols", "2008-09-11", "TX"),
    -> (3, 4, 2, "Hedda Kent", "2005-10-27", "TX"),
    -> (4, 2, 1, "Orli Strong", "2001-07-01", "NY"),
    -> (5, 1, 1, "Leonard Haynes", "2011-05-30", "MS"),
    -> (6, 1, 5, "Colette Payne", "2002-10-22", "MS"),
    -> (7, 3, 4, "Cooper Hatfield", "2010-08-19", "NY"),
    -> (8, 2, 4, "Timothy Battle", "2001-01-21", "NY"),
    -> (9, 3, 1, "Doris Munoz", "2008-10-22", "NY"),
    -> (10, 4, 2, "Alea Wiggins", "2007-08-21", "TX");
Query OK, 10 rows affected (0.23 sec)
Records: 10  Duplicates: 0  Warnings: 0

mysql> INSERT INTO salaries (employeeId, salary) VALUES
    -> (1, 885219), (2, 451519), (3, 288905), (4, 904312), (5, 919124),
    -> (6, 101538), (7, 355077), (8, 900436), (9, 41557), (10, 556263);
Query OK, 10 rows affected (0.23 sec)
Records: 10  Duplicates: 0  Warnings: 0
```

```sql
mysql> SELECT COUNT(*) from employees;
+----------+
| COUNT(*) |
+----------+
|       10 |
+----------+
1 row in set (0.19 sec)

mysql> SELECT id, name FROM employees ORDER BY id;
+----+----------------+
| id | name           |
+----+----------------+
|  1 | Karly Steele   |
|  2 | Rhona Nichols  |
|  3 | Hedda Kent     |
|  4 | Orli Strong    |
|  5 | Leonard Haynes |
|  6 | Colette Payne  |
|  7 | Cooper Hatfield|
|  8 | Timothy Battle |
|  9 | Doris Munoz    |
| 10 | Alea Wiggins   |
+----+----------------+
10 rows in set (0.29 sec)

mysql> SELECT id, name FROM employees WHERE state = 'TX' ORDER BY id;
+----+--------------+
| id | name         |
+----+--------------+
|  2 | Rhona Nichols|
|  3 | Hedda Kent   |
| 10 | Alea Wiggins |
+----+--------------+
3 rows in set (0.30 sec)

mysql> SELECT id, name FROM employees WHERE state = 'NY' ORDER BY id;
+----+---------------+
| id | name          |
+----+---------------+
|  1 | Karly Steele  |
|  4 | Orli Strong   |
|  7 | Cooper Hatfield|
|  8 | Timothy Battle|
|  9 | Doris Munoz   |
+----+---------------+
5 rows in set (0.15 sec)
```

## SingleStore vs. Traditional Databases

The following comparison table outlines the key differences between SingleStore and traditional databases:

| **Feature**               | **SingleStore**                                | **Traditional Databases**                                |
|---------------------------|------------------------------------------------|------------------------------------------------|
| **Speed and Performance**  | Ideal for real-time analytics and hybrid transactional/analytical processing (HTAP). | Traditional databases like Oracle and MySQL often struggle with performance under heavy loads.  |
| **Scalability**            | Supports horizontal scalability by adding nodes to handle large datasets. | Many traditional databases rely on vertical scaling, which can be expensive and limited. |
| **Flexibility**            | Offers deployment flexibility across on-premises, cloud, and containerized environments (e.g., Kubernetes). | Some databases have limited deployment options.  |
| **Cost Efficiency**        | Delivers high performance at a lower cost than enterprise solutions like Oracle. | Enterprise solutions typically come with high operational costs. Open-source databases like PostgreSQL or MySQL are more cost-effective but may require additional features to meet application demands. |

## Ideal Market

SingleStore is ideally suited for markets and industries that require high-performance and flexible database solutions. Below are key market segments where SingleStore provides value:

- **E-Commerce and Retail**: Managing transactions in real-time is crucial for online stores. SingleStore's capability to process large datasets in real-time ensures that businesses operate smoothly even during peak sales periods.
- **Healthcare and Life Sciences**: Managing patient records, analyzing clinical data, and supporting advanced research requires a database that is fast, secure, and scalable. SingleStore helps healthcare organizations manage their data effectively.
- **Gaming and Media**: In the fast-paced world of gaming and media, where technology constantly advances, SingleStore provides low latency and high throughput, enhancing user interactions.

## Getting Involved

- Learn more about the tool on the [SingleStore](https://www.singlestore.com/) website.
- Join community events and learn more about [Generative AI](https://www.singlestore.com/resources/?category=webinar).
- Become an expert in SingleStore with [SingleStore Training](https://www.singlestore.com/training/).

## Conclusion

Databases are the backbone of any application, ensuring efficient data management. The choice of a database varies depending on specific use cases. This blog has covered SingleStore as an effective solution for deployment in cloud-native environments, particularly using a platform like Civo. SingleStore addresses the limitations of traditional databases by offering performance, security, scalability, and flexibility.

*Shoutout to [SingleStore](https://msql.co/deploying-genai) for collaborating with me on this post.*
