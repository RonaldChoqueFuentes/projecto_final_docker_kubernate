### Entregables Parte 1

```bash
microk8s status
```

![alt text](screenshoots/parte_01/image.png)

```bash
kubectl get all -n proyecto-integrador
```
![alt text](screenshoots/parte_01/image-1.png)

Localhost browser

![alt text](screenshoots/parte_01/image-2.png)

Virtual Box Configuration

![alt text](screenshoots/parte_01/image-3.png)

## Parte 2: Iteración v2.1 - Modificar Backend
### Objetivo
Agregar un nuevo endpoint en el backend, versionar la imagen como v2.1, publicarla en tu Docker Hub y actualizar el deployment.
### Tareas

#### 2.1 Agregar Nuevo Endpoint

Editar el archivo: `src/main/java/dev/alefiengo/api/controller/GreetingController.java`

Agregar el siguiente método:

```java
@GetMapping("/api/info")
public ResponseEntity<Map<String, Object>> getInfo() {
    Map<String, Object> info = new HashMap<>();
    info.put("alumno", "TU NOMBRE COMPLETO");
    info.put("version", "v2.1");
    info.put("curso", "Docker & Kubernetes - i-Quattro");
    info.put("timestamp", LocalDateTime.now().toString());
    info.put("hostname", System.getenv("HOSTNAME"));
    return ResponseEntity.ok(info);
}
```

**Importante:** Reemplazar "TU NOMBRE COMPLETO" con tu nombre real.

#### 2.2 Build Imagen Docker v2.1

```bash
# Build imagen (reemplaza 'tu-usuario' con tu username de Docker Hub)
docker build -t tu-usuario/springboot-api:v2.1 .

# Verificar imagen
docker images | grep springboot-api
```

**Nota:** Ya debes estar logueado en Docker Hub desde el Paso 1.4.

#### 2.3 Push a Docker Hub

```bash
# Push
docker push tu-usuario/springboot-api:v2.1

# Verificar en https://hub.docker.com/r/tu-usuario/springboot-api/tags
```

#### 2.4 Actualizar Deployment de Kubernetes

Editar: `k8s/05-backend/api-deployment.yaml`

Buscar y cambiar la imagen del contenedor `api`:
```yaml
# Antes:
image: alefiengo/springboot-api:v2.0

# Después:
image: tu-usuario/springboot-api:v2.1
```

#### 2.5 Aplicar Cambios

```bash
# Aplicar el deployment actualizado
kubectl apply -f k8s/05-backend/api-deployment.yaml

# Ver el estado del rollout
kubectl rollout status deployment/api -n proyecto-integrador

# Ver los pods actualizándose
kubectl get pods -n proyecto-integrador -w
```

**Observación:** Kubernetes hará un rolling update automático. Verás pods nuevos con v2.1 creándose y los viejos terminándose gradualmente.

**Nota:** Como usaste un tag nuevo (v2.1 en lugar de v2.0), Kubernetes detecta el cambio automáticamente y descarga la nueva imagen. No necesitas limpiar caché manualmente.

**¿Y si la imagen no se actualiza?** Ver la sección del FAQ: "¿Por qué Kubernetes no actualiza mi imagen después de hacer push a Docker Hub?"

#### 2.6 Verificar Funcionamiento

```bash
# Opción 1: Port-forward
kubectl port-forward -n proyecto-integrador svc/api-service 8080:8080

# En otra terminal
curl http://localhost:8080/api/info

# Opción 2: Via Ingress
curl http://<IP-METALLB>/api/info
```

**Salida esperada:**
```json
{
  "alumno": "Tu Nombre",
  "version": "v2.1",
  "curso": "Docker & Kubernetes - i-Quattro",
  "timestamp": "2025-01-17T10:30:00",
  "hostname": "api-xxxx-yyyy"
}
```

#### 2.7 Crear Tag en Git

```bash
git add .
git commit -m "feat: add info endpoint for v2.1"
git tag -a v2.1 -m "Backend v2.1 con endpoint /api/info"
```

**ACCIÓN REQUERIDA:** Una vez que hayas verificado que el endpoint `/api/info` funciona correctamente, captura los screenshots solicitados en los Entregables Parte 2.

### Entregables Parte 2
- Código del endpoint agregado (screenshot o archivo .java)
- Screenshot de `docker images` mostrando la imagen v2.1
- Link a tu imagen en Docker Hub: `https://hub.docker.com/r/tu-usuario/springboot-api/tags`
- Screenshot de `kubectl rollout status` durante la actualización
- Screenshot de `kubectl get pods` mostrando los pods con la nueva versión
- Screenshot o output de `curl http://<IP-METALLB>/api/info` mostrando la respuesta JSON

---