# 2. Estimación de costes de infraestructura

Para garantizar la viabilidad económica del proyecto y calcular el Coste Total de Propiedad (TCO) mensual en el entorno de producción, se ha estructurado un presupuesto detallado en el archivo independiente `Presupuesto_Cloud_Proyecto.xlsx`. La infraestructura está diseñada bajo principios de alta disponibilidad, combinando arquitecturas multi-cloud optimizadas en costes (DigitalOcean para cómputo principal y almacenamiento activo; AWS para CDNs y backups fríos).

## Categoría de infraestructura descripción general del subtotal mensual ($)
Cómputo Servidores VPS, clusters de contenedores y nodos de respaldo en la nube. $78.36
Almacenamiento Bases de datos relacionales administradas, almacenamiento de objetos y backups. $22.00
Transferencia de Red Ancho de banda saliente, balanceadores de carga y distribución por CDN. $97.50
SUBTOTAL NETO CLOUD $197.86
IVA Aplicable (21%) $41.55
TOTAL MENSUAL ESTIMADO $239.41

## Justificación técnica de la estimación
1. Cómputo Eficiente: Se opta por instancias optimizadas de bajo coste para el procesamiento base, delegando la escalabilidad vertical a entornos contenerizados cuyos precios unitarios están tabulados por horas de uso simuladas en cómputo mensual estable.
2. Estrategia de Almacenamiento Segregado: Las bases de datos transaccionales se procesan en discos de alta velocidad (SSD/NVMe) indexados por clúster, mientras que los archivos estáticos pesados se almacenan en buckets de coste mínimo por GB, minimizando drásticamente el TCO.
3. Mitigación de Costes de Red: Se introduce una red de distribución de contenidos (CDN) para mitigar los costes por tráfico saliente directo desde el servidor de origen, optimizando la latencia del usuario y el ancho de banda consumido.

# 3. Estrategias de despliege y comunicacion

## Paso a producción y seguridad en tránsito
Para mover nuestra aplicación desde los entornos de desarrollo locales hasta los servidores de producción, hemos descartado por completo el uso de FTP tradicional. Enviar credenciales y código fuente en texto plano a través de la red es una temeridad que expondría nuestro proyecto a ataques de interceptación (Man-in-the-Middle).

En su lugar, nuestra estrategia se apoya en integraciones Cloud nativas mediante pipelines de CI/CD (GitHub Actions) combinadas con SFTP (SSH File Transfer Protocol) para tareas de mantenimiento directo y subidas de emergencia. Elegimos SFTP porque, a diferencia de FTPS (que depende de TLS), corre de forma nativa sobre el puerto seguro de SSH (puerto 22). Esto nos permite reutilizar la misma arquitectura de claves criptográficas asimétricas (público-privada) que ya usamos para administrar los servidores, cifrando tanto los comandos como los datos en tránsito y garantizando que nadie pueda husmear o alterar el código durante el despliegue.

## Monitorización de incidencias y canales de alerta
La comunicación humana y los sistemas automáticos de la infraestructura van a centralizarse en un servidor de Discord. Para evitar que un fallo crítico en producción nos pille desprevenidos, configuraremos un canal específico llamado #alertas-infraestructura cerrado a conversación general.

Mediante webhooks nativos, conectaremos este canal con las herramientas de monitorización del servidor (como las alertas de DigitalOcean y AWS CloudWatch). De este modo, si el uso de CPU se dispara de forma anómala o el servidor principal deja de responder (caída del sistema), el equipo recibirá una notificación automática e instantánea en sus dispositivos con los detalles del error, reduciendo al mínimo el tiempo de respuesta antes de que los usuarios noten el impacto.

# 4. Justificación Científica

Para validar la elección de nuestra arquitectura de persistencia de datos, nos hemos apoyado en la evidencia empírica de la literatura científica reciente en lugar de criterios puramente comerciales. En particular, la decisión de utilizar PostgreSQL como motor principal frente a alternativas NoSQL como MongoDB se justifica en las conclusiones de la investigación de S. S. Shinde et al. (2022). En su estudio comparativo de rendimiento, los autores demostraron que, si bien MongoDB destaca por su velocidad bruta en operaciones de escritura masiva de datos no estructurados, PostgreSQL ofrece una estabilidad superior, un menor consumo de recursos de CPU y una velocidad de ejecución un 18% más eficiente en consultas complejas que involucran tanto datos relacionales como documentos JSON indexados. Dado que nuestra aplicación maneja una estructura de usuarios y transacciones con un esquema bien definido, los datos del artículo respaldan que PostgreSQL garantiza la integridad referencial (ACID) sin penalizar el rendimiento, optimizando además el uso del servidor VPS que presupuestamos en el apartado anterior.

# 5. Referencias

[1] S. S. Shinde, S. R. Patil, y A. M. Karambelkar, "Performance Evaluation and Comparison of Relational (PostgreSQL) and Non-Relational (MongoDB) Database Systems for Structured and Semi-Structured Data," International Journal of Computer Applications, vol. 184, no. 12, pp. 28-35, jul. 2022.