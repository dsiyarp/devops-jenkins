
# Guía Paso a Paso: Configuración de Jenkins, GitHub y Apache en Máquinas Virtuales

## Preparar el Entorno

### 1. Configurar las Máquinas Virtuales

- **VM1**: Instalar y configurar Jenkins y Git.
- **VM2**: Instalar Apache Web Server y OpenSSH.

## 2. Instalar Jenkins en VM1

1. Conéctate a **VM1** usando SSH o la terminal de VirtualBox.
2. Instala Jenkins con los siguientes comandos:

   ```bash
   sudo apt update
   sudo apt install openjdk-11-jdk -y
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb http://pkg.jenkins.io/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
   sudo apt update
   sudo apt install jenkins -y
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```

3. Verifica que Jenkins esté corriendo:

   ```bash
   sudo systemctl status jenkins
   ```

## 3. Exponer Jenkins con Ngrok

1. Instala Ngrok en **VM1**:

   ```bash
   sudo apt install unzip -y
   wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
   unzip ngrok-stable-linux-amd64.zip
   sudo mv ngrok /usr/local/bin
   ```

2. Inicia Ngrok para exponer Jenkins:

   ```bash
   ngrok http 8080
   ```

3. Copia la URL pública proporcionada por Ngrok (algo como `https://abcd1234.ngrok.io`).

## 4. Acceder a Jenkins

1. Abre un navegador y navega a la URL de Ngrok.
2. Sigue las instrucciones para desbloquear Jenkins y crear un usuario administrador.

## Configurar VM2 con OpenSSH

### 5. Instalar OpenSSH en VM2

1. Conéctate a **VM2**.
2. Instala OpenSSH:

   ```bash
   sudo apt update
   sudo apt install openssh-server -y
   sudo systemctl start ssh
   sudo systemctl enable ssh
   ```

3. Verifica que el servicio esté corriendo:

   ```bash
   sudo systemctl status ssh
   ```

### 6. Generar y Configurar Llaves SSH

1. En **VM1**, genera una clave SSH:

   ```bash
   ssh-keygen -t rsa -b 4096 -C "usuario@example.com"
   ```

2. Copia la clave pública a **VM2**:

   ```bash
   ssh-copy-id usuario@ip_vm2
   ```

3. Verifica la conexión:

   ```bash
   ssh usuario@ip_vm2
   ```

## 7. Configurar Credenciales Globales en Jenkins

1. En Jenkins, ve a `Manage Jenkins` > `Manage Credentials`.
2. Añade credenciales SSH usando la clave privada generada en **VM1**.

## Configurar un Agent en Jenkins

### 8. Añadir un Nuevo Nodo en Jenkins

1. Ve a `Manage Jenkins` > `Manage Nodes and Clouds`.
2. Añade un nuevo nodo (Agent) usando la conexión SSH a **VM2**.
3. Configura la conexión usando las credenciales creadas.

## Configurar GitHub y Webhooks

### 9. Crear un Repositorio en GitHub

1. Crea un nuevo repositorio en GitHub.
2. Clona el repositorio en tu máquina local.

### 10. Configurar el Webhook en GitHub

1. Ve a `Settings` > `Webhooks` en GitHub.
2. Añade un webhook con la URL de Jenkins expuesta por Ngrok (`https://abcd1234.ngrok.io/github-webhook/`).

## Crear el Pipeline de Jenkins

### 11. Crear un Jenkinsfile en el Repositorio

Crea un archivo llamado `Jenkinsfile` con el siguiente contenido:

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git url: 'git@github.com:usuario/repo.git', credentialsId: 'credenciales_id', branch: 'main'
            }
        }
        stage('Build') {
            steps {
                echo 'Compilando el proyecto...'
            }
        }
        stage('Deploy') {
            steps {
                sh 'scp -r . usuario@ip_vm2:/var/www/html/'
            }
        }
    }
}
```

### 12. Hacer Commit y Push del Jenkinsfile

1. Realiza el commit y push del archivo `Jenkinsfile`:

   ```bash
   git add Jenkinsfile
   git commit -m "Añadir Jenkinsfile"
   git push origin main
   ```

## Ejecución y Prueba

### 13. Realizar un Commit en GitHub

1. Modifica un archivo (por ejemplo, `index.html`) y haz commit y push al repositorio.

### 14. Verificar el Despliegue en Apache

1. Jenkins debería detectar el commit y ejecutar el pipeline automáticamente.
2. Navega a `http://ip_vm2/` para verificar que el contenido se haya desplegado correctamente en **VM2**.

## Problemas Comunes

- **Error de verificación de clave de host**: Ejecuta en **VM1**:
  ```bash
  ssh-keyscan ip_vm2 >> ~/.ssh/known_hosts
  ```

- **Permisos denegados en `scp`**: Asegúrate de que el usuario tenga permisos de escritura en `/var/www/html/`:
  ```bash
  sudo chown -R usuario:usuario /var/www/html/
  ```

## Conclusión

Esta guía debería permitirte configurar un pipeline en Jenkins que clona el repositorio de GitHub, compila el proyecto y despliega los archivos en el servidor Apache de **VM2**.
