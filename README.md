# Micro-frontends in Kubernetes

Some YAML files used while exploring options to implement micro-frontend architectures in Kubernetes and with Ingress Nginx controller.
This YAMLs are the companion resources for an article I wrote in my personal blog: [Micro-frontends and Ingress Nginx - Server Side Includes](https://ruben-rodriguez.github.io/posts/ingress-nginx-server-side-includes/)

For testing out the different configuration scenarios, I recommend using a local K8s cluster, using, for example, [kind](https://kind.sigs.k8s.io/) 

## The Good - micro-frontend behind Ingress

YAML files and configuration for this setup can be found in the `micro-frontend-behind-ingress` folder of this repository.
For this option, we will need a standard Ingress Nginx installation:

```
λ  helm upgrade --install ingress-nginx ingress-nginx \
          --repo https://kubernetes.github.io/ingress-nginx \
          --namespace ingress-nginx --create-namespace
```

After the Nginx ingress controller is installed, you can create the services stack and the Ingress object:

```
λ  kubectl apply -f stack.yaml
λ  kubectl apply -f ingress.yaml
```

After test, clean up with:

```
λ  kubectl delete -f ingress.yaml
λ  kubectl delete -f stack.yaml
λ  helm uninstall ingress-nginx -n ingress-nginx
```

## The Evil - micro-frontend at Ingress with server-snippets

YAML files and configuration for this setup can be found in the `micro-frontend-at-ingress` folder of this repository.
For this option, we will need to install Ingress Nginx with support for `server-snippets`:
```
λ  helm upgrade --install ingress-nginx ingress-nginx \
          --repo https://kubernetes.github.io/ingress-nginx \
          --namespace ingress-nginx --create-namespace \
          --set controller.allowSnippetAnnotations=true
```

After the Nginx ingress controller is installed, you can create the services stack and the Ingress object:

```
λ  kubectl apply -f stack.yaml
λ  kubectl apply -f ingress.yaml
```

You can also test the exploit for [CVE-2021-25742](https://github.com/kubernetes/ingress-nginx/issues/7837):

```
λ  kubectl apply -f exploit-ingress.yaml
```

Finally, clean up with:

```
λ  kubectl apply -f stack.yaml
λ  kubectl apply -f ingress.yaml
λ  helm uninstall ingress-nginx -n ingress-nginx
```

## The Ugly - micro-frontend at Ingress with custom configuration

YAML files and configuration for this setup can be found in the `micro-frontend-at-ingress-custom-config` folder of this repository.
For this option, we will need to install Ingress Nginx with the appropriate values to use the custom configuration (ConfigMap) that has `ssi on;` directive:
```
λ  kubectl apply -f ingress-custom-template.yaml
λ  helm upgrade --install ingress-nginx ingress-nginx \
          --repo https://kubernetes.github.io/ingress-nginx \
          --namespace ingress-nginx --create-namespace \
          --set controller.customTemplate.configMapName=ingress-nginx-template \
          --set controller.customTemplate.configMapKey=main-template
```
Then, create the services stack and the Ingress object:

```
λ  kubectl apply -f stack.yaml
λ  kubectl apply -f ingress.yaml
```

Finally, clean up with:

```
λ  kubectl apply -f stack.yaml
λ  kubectl apply -f ingress.yaml
λ  helm uninstall ingress-nginx -n ingress-nginx
```
