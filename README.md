NOTE: When starting "skaffold dev" if one encounters the following error then that means that minikube is NOT running.  For minikube use the podman driver. The docker driver is NOT compatible with ingress.
"skaffold KUBERNETES_SERVICE_HOST and KUBERNETES_SERVICE_PORT must be defined"

minikube start --driver podman
...
minikube stop

$ minikube start
😄  minikube v1.16.0 on Ubuntu 18.04
✨  Using the podman (experimental) driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🔄  Restarting existing podman container for "minikube" ...
🐳  Preparing Kubernetes v1.20.0 on Docker 20.10.0 ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

-------------
NOTE: Here is the discussion related to docker driver not being compatible with ingress:
https://www.udemy.com/course/microservices-with-node-js-and-react/learn/lecture/23145358#questions

-------------
NOTE: ingress must also be enabled for the communication to work

$ minikube addons enable ingress
❗  Executing "sudo -n podman container inspect minikube --format={{.State.Status}}" took an unusually long time: 2.200250535s
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled

PS: https://kubernetes.github.io/ingress-nginx/deploy/

-------------
NOTE: Test the ingress install: (Note the new ingress-nginx pods)

$ kubectl get pods -n kube-system
NAME                                        READY   STATUS      RESTARTS   AGE
coredns-74ff55c5b-xvm5d                     1/1     Running     2          29h
etcd-minikube                               1/1     Running     2          29h
ingress-nginx-admission-create-8xvkn        0/1     Completed   0          4m24s
ingress-nginx-admission-patch-rtdf5         0/1     Completed   2          4m24s
ingress-nginx-controller-558664778f-w767p   1/1     Running     0          4m26s
kube-apiserver-minikube                     1/1     Running     2          29h
kube-controller-manager-minikube            1/1     Running     2          29h
kube-proxy-82h64                            1/1     Running     2          29h
kube-scheduler-minikube                     1/1     Running     2          29h
storage-provisioner                         1/1     Running     5          29h

$ kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
deployment.apps/web created

$ kubectl expose deployment web --type=NodePort --port=8080
service/web exposed

$ kubectl get service web
NAME   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
web    NodePort   10.100.205.91   <none>        8080:31935/TCP   13s

!!!NOTE: The special 3xxxx port that is generated for minikube

$ minikube service web --url
http://192.168.59.2:31935

$ curl http://192.168.59.2:31935
Hello, world!
Version: 1.0.0
Hostname: web-79d88c97d6-8mjd7

-------------
NOTE: React warnings:  If one sees a compiled warning like the following, then it can be ignored or fixed.
src/App.js
[client]   Line 5:1:  Assign arrow function to a variable before exporting as module default  import/no-anonymous-default-export

Solution to above React compile warning: (NOTE: the function name must be capitalized)
Instead of:
export default () => { ... };

use:
const Fn = () => { ... };
...
export default Fn;

-------------
NOTE1: Before "skaffold dev" is started the client Dockerfile needs to be modified to include the line:
"ENV CI=true"

FROM node:alpine

ENV CI=true

WORKDIR /app
COPY package.json ./
RUN npm install
COPY ./ ./

CMD ["npm", "start"]

NOTE2: If a change needed to be made, then do the following:
docker build -t ms2021email/client .
docker push ms2021email/client

The build and push also needs to be done for the other React apps: (Need to cd into respective dir)

docker build -t ms2021email/comments .
docker push ms2021email/comments

docker build -t ms2021email/event-bus .
docker push ms2021email/event-bus

docker build -t ms2021email/moderation .
docker push ms2021email/moderation

docker build -t ms2021email/posts .
docker push ms2021email/posts

docker build -t ms2021email/query .
docker push ms2021email/query

cd into infra/k8s directory:
kubectl apply -f client-depl.yaml 
kubectl apply -f comments-depl.yaml 
kubectl apply -f event-bus-depl.yaml 
kubectl apply -f moderation-depl.yaml 
kubectl apply -f posts-depl.yaml 
kubectl apply -f query-depl.yaml 

kubectl apply -f ingress-srv.yaml 

Note1: if the system is already running, then may need to do the following as well to reload the services:
kubectl rollout restart deployment client-depl
kubectl rollout restart deployment comments-depl
kubectl rollout restart deployment event-bus-depl
kubectl rollout restart deployment moderation-depl
kubectl rollout restart deployment posts-depl
kubectl rollout restart deployment query-depl

Note2: If using "skaffold dev" then the above is automatically done, instead of needing to be done manually.

-------------

NOTE: Modify the /etc/hosts internal IP address used for the URL posts.com

$ minikube ip
❗  Executing "sudo -n podman container inspect minikube --format={{.State.Status}}" took an unusually long time: 2.291271934s
192.168.59.2

# Set ip address in the /etc/hosts file:
192.168.59.2  posts.com

-------------------

NOTE1: If above ip address is correct then the browser output should NOT show something like:
posts.com
...
Welcome to nginx!
...

NOTE2: It should show something like the following instead:
curl http://posts.com
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <link rel="apple-touch-icon" href="/logo192.png" />
    <!--
      manifest.json provides metadata used when your web app is installed on a
      user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
    -->
    <link rel="manifest" href="/manifest.json" />
    <link
      rel="stylesheet"
      href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css"
      integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh"
      crossorigin="anonymous"
    />

    <!--
      Notice the use of  in the tags above.
      It will be replaced with the URL of the `public` folder during the build.
      Only files inside the `public` folder can be referenced from the HTML.

      Unlike "/favicon.ico" or "favicon.ico", "/favicon.ico" will
      work correctly both with client-side routing and a non-root public URL.
      Learn how to configure a non-root public URL by running `npm run build`.
    -->
    <title>React App</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
    <!--
      This HTML file is a template.
      If you open it directly in the browser, you will see an empty page.

      You can add webfonts, meta tags, or analytics to this file.
      The build step will place the bundled scripts into the <body> tag.

      To begin the development, run `npm start` or `yarn start`.
      To create a production bundle, use `npm run build` or `yarn build`.
    -->
  <script src="/static/js/bundle.js"></script><script src="/static/js/vendors~main.chunk.js"></script><script src="/static/js/main.chunk.js"></script></body>
</html>

-------------------
NOTE: If one is not able to access the application "posts.com" then one may already have something running on port 80, which is the default port for ingress.

For macOS/linux use the following command to check:
sudo lsof -i tcp:80

For windows: (windows does NOT have grep but does have findstr command)
netstat -aon | findstr :80
-------------------

Limitation of Ingress Controller:  It cannot do routing based on the method of the request like GET/POST.
So Ingress can only route based on paths, so each path must be unique.

-------------------

NOTE: The "skaffold dev" output should look like the following:
...
[client] > client@0.1.0 start
[client] > react-scripts start
[client] 
[client] ℹ ｢wds｣: Project is running at http://172.17.0.4/
[client] ℹ ｢wds｣: webpack output is served from 
[client] ℹ ｢wds｣: Content not from webpack is served from /app/public
[client] ℹ ｢wds｣: 404s will fallback to /
[client] Starting the development server...
[client] 
[client] Compiled successfully!
[client] 
[client] You can now view client in the browser.
[client] 
[client]   Local:            http://localhost:3000
[client]   On Your Network:  http://172.17.0.4:3000
[client] 
[client] Note that the development build is not optimized.
[client] To create a production build, use npm run build.

--------------

Concerning the React app:
- each separate node app services (client, comments, event-bus, moderation, posts, query) uses "nodemon index.js" which automatically restarts the service/app anytime that it sees a change to a file inside of a given directory. This results in the browser automatically being updated.

- also Skaffold automatically rebuilds the docker image and redeploys the container inside of the pod anytime that it sees a change to a file, which automatically keeps the images/contianers/pods upto date.  It however does NOT check the new image into docker hub.

- There are a few circumstances that nodemon or skaffold might have trouble detecting a change to a file inside of a container.  So if this happens then one may need to manually restart "skaffold dev" as a worst case scenario.

--------------

NOTE: If one does not use "skaffold dev", then there are manual steps for each piece of the app.  Plus this assumes that minikube is already running, and the infra/k8s deployment and service yaml files have already been created.

- docker build -t ms2021email/client .
- docker push ms2021email/client
- kubectl rollout restart deployment client-depl

PS: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

--------------

Note: If one was using minikube without "skaffold dev", then do the following to use "skaffold dev"

$ minikube stop
✋  Stopping node "minikube"  ...
🛑  Powering off "minikube" via SSH ...
✋  Stopping node "minikube"  ...
🛑  1 nodes stopped.

$ minikube delete
🔥  Deleting "minikube" in podman ...
🔥  Deleting container "minikube" ...

# Delete all of the running containers
docker rm $(docker ps -a -q)

minikube start --driver podman

minikube addons enable ingress

#cd to directory with skaffold file
skaffold dev

--------------
