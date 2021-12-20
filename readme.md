**Install istio from code.From base directory, run:**
```bash
go run istioctl/cmd/istioctl/main.go install
```

**Install examples**
From the base directory, run
https://istio.io/latest/docs/examples/bookinfo/

Follow all step thru Gateway and access the product page on browser:
```
minikube service productpage
```
and replace the browser URL with
```
http://Host:NodePort/productpage
```

**Build and Debug**
[Hack]
From Istio base directory, create this
mkdir -p ./out/linux_amd64/docker_build/docker.pilot/

a). Add this in pilot/docker/Dockerfile.pilot
```docker
#ENTRYPOINT ["/usr/local/bin/pilot-discovery"]

EXPOSE 40000
COPY ./dlv/dlv /usr/local/bin/dlv
ENTRYPOINT ["/usr/local/bin/dlv", "--listen=:40000", "--headless=true", "--api-version=2", "exec", "/usr/local/bin/pilot-discovery", "--"]
```

b). Copy dlv in ./out/linux_amd64/docker_build/docker.pilot/


1. Build pilot 
```bash
sudo GOOS=linux DEBUG=1 make docker.pilot
```

2. Export istio/pilot image to tar if we using local machine Docker daemon. 
Grab the image id.
```
docker save <IMG ID> -o istio_pilot.tar
```

3. ssh to minikube 
```bash
minikube ssh
```
or switch minikube daemon
```
eval $(minikube docker-env)
```

4. Import the image into minikube Docker
```bash
docker load -i istio_pilot.tar
```

5. Tag the newly import image
docker tag <Imported Image ID> my-local/istio_pilot:latest   


6. Change IstioD deployment. Update image, pullPolicy and expose DEBUG Port
```yaml
   image: <IMAGE>
   imagePullPolicy: Never
   ....
       - containerPort: 40000
         protocol: TCP  
```yaml         

You might have tweak securityContext for dlv. Change this in IstioD deploymet:
```yaml
      securityContext:
            capabilities:
              drop:
                - ALL
            runAsUser: 1337
            runAsGroup: 1337
            runAsNonRoot: true
            readOnlyRootFilesystem: false
            allowPrivilegeEscalation: true         
```            

1. Setup port forwarding 40000 from isioD

2. Add Intellij remote debug wit 40000