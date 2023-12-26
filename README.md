# Netflix-Clone deployed docker & k8's.

**Step 1: Clone the repo into you system:**

```bash
git clone https://github.com/Vikramop/netflix-clone.git
```

**Step 1: Launch Docker Desktop and minkube:**

- If you don not have a [Docker Desktop](https://www.docker.com/products/docker-desktop/) then Download for your system.
- Then go top settings->kubernetes->enable kubernetes.
- Set up a [minikube](https://minikube.sigs.k8s.io/docs/start/) cluster.
- Then make a account on [dockerhub](https://hub.docker.com/).
- After all set up _let's start_.

**Step 2: Let's create a Image of our app on dockerhub**

```bash
docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .
```

- Replace your API key with TMDB api by login->setting->API

```
[+] Building 51.9s (19/19) FINISHED                                                                                                             docker:default
 => [internal] load .dockerignore                                                                                                                         0.4s
 => => transferring context: 61B                                                                                                                          0.4s
 => [internal] load build definition from Dockerfile                                                                                                      0.4s
 => => transferring dockerfile: 453B                                                                                                                      0.4s
 => [internal] load metadata for docker.io/library/node:16.17.0-alpine                                                                                    2.5s
 => [internal] load metadata for docker.io/library/nginx:stable-alpine                                                                                    3.3s
 => [auth] library/nginx:pull token for registry-1.docker.io                                                                                              0.0s
 => [auth] library/node:pull token for registry-1.docker.io                                                                                               0.0s
 => [stage-1 1/4] FROM docker.io/library/nginx:stable-alpine@sha256:089520833b93077841d3cdc7ab1f7b817de73c7e10070b71b85fa97da7623dbe                      0.0s
 => [builder 1/7] FROM docker.io/library/node:16.17.0-alpine@sha256:2c405ed42fc0fd6aacbe5730042640450e5ec030bada7617beac88f742b6997b                      0.0s
 => [internal] load build context                                                                                                                         0.1s
 => => transferring context: 8.90kB                                                                                                                       0.1s
 => CACHED [builder 2/7] WORKDIR /app                                                                                                                     0.0s
 => CACHED [builder 3/7] COPY ./package.json .                                                                                                            0.0s
 => CACHED [builder 4/7] COPY ./yarn.lock .                                                                                                               0.0s
 => CACHED [builder 5/7] RUN yarn install                                                                                                                 0.0s
 => [builder 6/7] COPY . .                                                                                                                                0.4s
 => [builder 7/7] RUN yarn build                                                                                                                         42.6s
 => CACHED [stage-1 2/4] WORKDIR /usr/share/nginx/html                                                                                                    0.0s
 => CACHED [stage-1 3/4] RUN rm -rf ./*                                                                                                                   0.0s
 => [stage-1 4/4] COPY --from=builder /app/dist .                                                                                                         0.8s
 => exporting to image                                                                                                                                    0.2s
 => => exporting layers                                                                                                                                   0.2s
 => => writing image sha256:2be1471d9acc740a02c12f9ff0501745d07adee9d6c7bde65ef8cd3d4457de68                                                              0.0s
 => => naming to docker.io/library/netflix-clone                                                                                                          0.0s
```

- This is how it looks.
- Push the Image on docker hub by:

```bash
docker push <username>/netflix-clone
```

- Replace `<username>` with your username from docker hub.

- we can also host on our local host by:

```bash
docker run -d --name netflix-clone -p 8081:80
```

- This will host on local:8081.

**Step 3: Create Kubernetes Deployment and Service**

- We go to the kubernetes folder in app directories.

```
cd kubernetes
```

- Now Let's create deployments and service.

```bash
kubectl apply -f deployment.yml

```

- Below is the output.

```
$ kubectl apply -f deployment.yml
deployment.apps/netflix-app created
```

- Same for service file

```
kubectl apply -f service.yml
```

- Run the below command

```
kubectl get pods
```

- And if you get ...

```
NAME                                 READY   STATUS             RESTARTS      AGE
netflix-app-6f4cc4b8cb-bj8bw         0/1     ErrImagePull       0             6m40s
netflix-app-6f4cc4b8cb-lkzxv         0/1     ImagePullBackOff   0             6m40s
```

- Then either of these cases have occured:docker is not running or the minikube is not started or the iamge is not present on docker hub or any typo in the image name(like nertflixx).

- After fixing the result will be like the below one.

```
NAME                                 READY   STATUS             RESTARTS      AGE
netflix-app-6f4cc4b8cb-bj8bw         1/1     Running            0             13m
netflix-app-6f4cc4b8cb-lkzxv         1/1     Running            0             13m
```

- Then run the below command to check the service is running or not.

```
kubectl get services
```

- The output must be like:

```
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes           ClusterIP   10.96.0.1        <none>        443/TCP          8d
netflix-app          NodePort    10.97.7.152      <none>        80:30007/TCP     10m
```

- Here we can see all the services running the first one is default and the second one is we deployed.

**Step 3: Hosting on your local server**

- So the ip will be combination of minikube ip and the ip that is prvided by kubectl.
- Run the following

```
minikube ip
```

- To get the minikube ip:`192.168.49.2`, in my case.
- Then combine the ip of minikube and netflix-clone's ip.
- Which looks like `192.168.49.2:30007` in my case.
