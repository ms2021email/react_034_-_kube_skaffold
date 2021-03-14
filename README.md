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
NOTE: Before "skaffold dev" is started the client Dockerfile needs to be modified to include the line:
"ENV CI=true"


FROM node:alpine

ENV CI=true

WORKDIR /app
COPY package.json ./
RUN npm install
COPY ./ ./

CMD ["npm", "start"]

-------------
$ minikube ip
❗  Executing "sudo -n podman container inspect minikube --format={{.State.Status}}" took an unusually long time: 2.291271934s
192.168.59.2

# Set ip address in the /etc/hosts file:
192.168.59.2  posts.com

-------------------

If above ip address is correct then the browser output should show something like:
posts.com
...
Welcome to nginx!
...
-------------------
NOTE: If one is not able to access the application "posts.com/posts" then one may already have something running on port 80, which is the default port for ingress.

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

