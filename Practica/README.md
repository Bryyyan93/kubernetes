# Práctica: Despliegue de una aplicación en Kubernetes
## Objetivos Específicos
- **Crear un chart de Helm**: Desarrollar un chart de Helm que incluya todos los recursos necesarios para desplegar la aplicación en Kubernetes.
- **Configurar la Persistencia de Datos**: Implementar un mecanismo que asegure que los datos de la base de datos sean almacenados de manera duradera.
- **Gestionar Configuración Sensible**: Asegurar que las credenciales y configuraciones sensibles se manejen de forma segura, utilizando los recursos adecuados de Kubernetes y evitando cualquier tipo de información sensible en el repositorio.
- **Asegurar Alta Disponibilidad (HA)**: La aplicación deberá estar siempre disponible configurando un mínimo número de réplicas que asegure una alta disponibilidad.
- **Escalar la Aplicación de manera automática**: El despliegue de la aplicación debe ser escalable, permitiendo ajustar el número de réplicas en base al uso de CPU. Esta configuración debe estar expuesta en el chart para poder ajustarse mediante el fichero values.yaml. Por defecto la aplicación debe escalar cuando supere el 70% de uso de CPU.
- **Exponer la Aplicación al exterior**: Configurar los recursos necesarios para que la aplicación sea accesible desde fuera del clúster de Kubernetes.
- **Garantizar la resiliencia de la Aplicación**: La aplicación deberá poder recuperarse ante errores. Aplicar el mecanismo que nos permite reiniciar la aplicación cuando no está funcionando y evitar que llegue tráfico a los PODs que no responden correctamente.
- **Documentación**: Elaborar una documentación clara que explique cómo desplegar y gestionar la aplicación utilizando el chart de Helm creado.

## Cosas hechas
### Configurar la Persistencia de Datos
Para asegurar los datos de manera persitente se ha usado el manifesto`persistentvolumeclaim`.
- **db-data-persistentvolumeclaim**: Asegura los datos almacenados en la base de datos desde `db-deployment.yaml`.
- **db-data-persistentvolumeclaim**: Asegura los datos almacenados en la base de datos desde `web-deployment.yaml`.
### Gestionar Configuración Sensible
Para realizar esto se ha usado el manifesto de `secret` y se usa dos:
- **db-secret.yaml**: Aqui se almacenan las variables de configuración para la base de datos, se usarán en los deployments de `db-deployment.yaml` y `web-deployment.yaml`.
- **hub-secret.yaml**: Aqui se almacenan las variables de configuración para descargaar las imagenes del repositorio privado de Docker Hub, se usarán en los deployments de `db-deployment.yaml` y `web-deployment.yaml`.
- **env-test1-env-configmap.yaml**: Aqui se almacenan las variables de configuración no criticas (contraseñas) para la base de datos, se usarán en los deployments de `db-deployment.yaml` y `web-deployment.yaml`.
### Escalar la Aplicación de manera automática
Para realizar la escabilidad se ha usado el manifesto de `web-hpa.yaml`.
Para realizar las pruebas se ha hecho de la siguiente manera:
- Comporbar que esta habilitado el servicio de metrics `minikube addons enable metrics-server` 
- Comprobar que se esta ejecutando: `kubectl get hpa -w`
- Aumentar la carga de trabajo para verificar el autoescalado `kubectl run load-generator-manual --image=busybox --command -- sh -c 'while true; do wget -q -O- http://nginx-hpa >dev/null 2>&1; done'`
### Exponer la Aplicación al exterior
Se hace uso del manifiesto del tipo `Ingress` el cual llamamos `web-ingress.yaml`.
Para añadir contenido a la aplicacion Flask se usará la siguiente petición:
``` 
curl -X POST -H "Content-Type: application/json" -d '{"title":"Clase de Docker", "description":"Aprender a crear y manejar contenedores"}' http://localhost:5000/notes
```

