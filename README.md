
### Varias Formas de Acceder a los Pods

Una vez que los pods están desplegados y el servicio está configurado, hay varias maneras de acceder a ellos:

1. **A través del Servicio (Service)**
   - Si estás utilizando Minikube, puedes obtener la IP del servicio usando `minikube service`:
     ```bash
     minikube service terraform-example-service -n k8s-ns-by-tf
     ```
   - Si estás en un entorno diferente, puedes obtener la IP del servicio con `kubectl`:
     ```bash
     kubectl get service terraform-example-service -n k8s-ns-by-tf
     ```
     La salida te mostrará la IP y el puerto donde el servicio está expuesto.

2. **Accediendo Directamente a los Pods (Port Forwarding)**
   - Puedes usar `kubectl port-forward` para acceder a un pod específico:
     ```bash
     kubectl port-forward pod/terraform-example-7997bdd7d7-557r6 8080:80 -n k8s-ns-by-tf
     ```
     Esto hará que el pod sea accesible en `http://localhost:8080`.

3. **Usando una IP Externa (si el tipo de servicio es LoadBalancer)**
   - Si el servicio está configurado como `LoadBalancer` y tu proveedor de Kubernetes soporta IPs externas, puedes acceder a la aplicación a través de la IP externa asignada al servicio:
     ```bash
     kubectl get services terraform-example-service -n k8s-ns-by-tf
     ```
     Busca el campo `EXTERNAL-IP` en la salida del comando.

### Verificación

Después de aplicar los cambios y dependiendo de la forma que elijas para acceder a los pods, puedes verificar el acceso abriendo tu navegador web y accediendo a la dirección IP y el puerto correspondiente, o utilizando herramientas como `curl` para hacer solicitudes HTTP.

```bash
curl http://localhost:8080
```

Esto debería mostrarte la página predeterminada de Nginx o cualquier otra salida configurada en tus pods.
