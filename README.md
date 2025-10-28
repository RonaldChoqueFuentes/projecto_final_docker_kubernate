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

---

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
    info.put("alumno", "RONALD CHOQUE FUENTES");
    info.put("version", "v2.1");
    info.put("curso", "Docker & Kubernetes - i-Quattro");
    info.put("timestamp", LocalDateTime.now().toString());
    info.put("hostname", System.getenv("HOSTNAME"));
    return ResponseEntity.ok(info);
}
```


#### 2.2 Build Imagen Docker v2.1

```bash
# Build imagen (reemplaza 'tu-usuario' con tu username de Docker Hub)
docker build -t ronaldchoque/springboot-api:v2.1 .

# Verificar imagen
docker images | grep springboot-api
```

![alt text](screenshoots/parte_02/image.png)

#### 2.3 Push a Docker Hub

```bash
# Push
docker ronaldchoque/springboot-api:v2.1
# Verificar en https://hub.docker.com/r/ronaldchoque/springboot-api/tags
```
![alt text](screenshoots/parte_02/image-1.png)

![alt text]screenshoots/parte_02/image-2.png)

#### 2.4 Actualizar Deployment de Kubernetes

Editar: `k8s/05-backend/api-deployment.yaml`

Buscar y cambiar la imagen del contenedor `api`:
```yaml
# Antes:
image: alefiengo/springboot-api:v2.0

# Después:
image: ronaldchoque/springboot-api:v2.1
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

![alt text](screenshoots/parte_02/image-3.png)


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

![alt text](screenshoots/parte_02/image-4.png)

#### 2.7 Crear Tag en Git

```bash
git add .
git commit -m "feat: add info endpoint for v2.1"
git tag -a v2.1 -m "Backend v2.1 con endpoint /api/info"

```
### Entregables Parte 2

![alt text](screenshoots/parte_02/image-5.png)

```bash
docker images
```
![alt text](screenshoots/parte_02/image-6.png)


https://hub.docker.com/r/ronaldchoque/springboot-api/tags

![alt text](screenshoots/parte_02/image-7.png)

GET POTS
```bash
kubectl get pods -n proyecto-integrador -w
```
![alt text](screenshoots/parte_02/image-8.png)

- Screenshot o output de `curl http://<IP-METALLB>/api/info` mostrando la respuesta JSON
![alt text](screenshoots/parte_02/image-9.png)

Screenshot de `kubectl rollout status` durante la actualización

```bash
kubectl rollout status deployment/api -n proyecto-integrador
```
![rollout status](screenshoots/parte_02/image-10.png)
---
## Parte 3: Iteración v2.2 - Modificar Frontend (25%)

### Objetivo
Agregar funcionalidad en el frontend para consumir el nuevo endpoint `/api/info`, versionar como v2.2 y desplegar.

### Tareas

#### 3.1 Modificar Frontend Angular

Editar: `frontend/src/app/app.component.html`

Agregar después del botón "Registrar Usuario" (alrededor de la línea 30):

```html
<div class="form-group">
  <button (click)="getSystemInfo()" class="btn-primary">
    Ver Info del Sistema
  </button>
</div>

<div *ngIf="systemInfo" class="card info-section">
  <h3>Información del Sistema</h3>
  <p><strong>Alumno:</strong> {{ systemInfo.alumno }}</p>
  <p><strong>Versión:</strong> {{ systemInfo.version }}</p>
  <p><strong>Curso:</strong> {{ systemInfo.curso }}</p>
  <p><strong>Timestamp:</strong> {{ systemInfo.timestamp }}</p>
  <p><strong>Pod:</strong> {{ systemInfo.hostname }}</p>
</div>
```

Editar: `frontend/src/app/app.component.ts`

Agregar la propiedad y método:

```typescript
export class AppComponent implements OnInit {
  // ... propiedades existentes ...
  systemInfo: any = null;

  // ... métodos existentes ...

  getSystemInfo(): void {
    this.http.get('/api/info').subscribe({
      next: (data) => {
        this.systemInfo = data;
        this.success = 'Información del sistema cargada';
        setTimeout(() => this.success = null, 3000);
      },
      error: (err) => {
        this.error = 'Error al obtener información del sistema';
        console.error('Error:', err);
      }
    });
  }
}
```

#### 3.2 Build Imagen Frontend v2.2

```bash
cd frontend

# Build imagen
docker build -t tu-usuario/angular-frontend:v2.2 .

# Push
docker push tu-usuario/angular-frontend:v2.2
```

#### 3.3 Actualizar Deployment

Editar: `k8s/06-frontend/frontend-deployment.yaml`

Buscar y cambiar la imagen del contenedor `frontend`:
```yaml
# Antes:
image: alefiengo/angular-frontend:v2.0

# Después:
image: tu-usuario/angular-frontend:v2.2
```

#### 3.4 Aplicar Cambios

```bash
# Aplicar el deployment actualizado
kubectl apply -f k8s/06-frontend/frontend-deployment.yaml

# Ver el estado del rollout
kubectl rollout status deployment/frontend -n proyecto-integrador

# Ver rolling update en acción
kubectl get pods -n proyecto-integrador -l app=frontend -w
```

**Observación:** Al igual que con el backend, Kubernetes detecta el cambio de tag (v2.0 → v2.2) y actualiza automáticamente.

#### 3.5 Verificar Funcionamiento

Acceder desde el navegador a: `http://<IP-METALLB>/`

- Hacer clic en "Ver Info del Sistema"
- Verificar que se muestre la información correctamente
- Refrescar varias veces y observar que el `hostname` puede cambiar (load balancing entre pods)

**ACCIÓN REQUERIDA:** Una vez que hayas verificado que el frontend v2.2 muestra correctamente la información del sistema, captura los screenshots solicitados en los Entregables Parte 3.

### Entregables Parte 3
- Código modificado de Angular (screenshots de .html y .ts)
- Link a tu imagen en Docker Hub: `https://hub.docker.com/r/tu-usuario/angular-frontend/tags`
- Screenshot de `kubectl get pods -w` durante el rolling update del frontend
- Screenshot del navegador mostrando el botón "Ver Info del Sistema"
- Screenshot del navegador mostrando la información del sistema cargada

---