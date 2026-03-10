# cicd-templates
Este proyecto contiene los templates de los archivos necesarios para configurar la nueva pipeline de gitHub actions

## .gihub/workflows
Build: construcción de la imagen
Deploy: despliegue en el entorno seleccionado

## k8s
Values: sustituye conceptualmente al Jenkinsfile que tenía el proyecto anteriormente. 

En la mayoría de los casos, este fichero no se define desde cero, sino que es una traducción directa del Jenkinsfile existente, reutilizando los valores que ya estaban definidos allí. 