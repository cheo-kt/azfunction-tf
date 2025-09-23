### **Infraestructura como código**

- **Utilizar archivos de definición**: Todas las herramientas de infraestructura como código tienen un formato propio para definir la infraestructura.
- **Autodocumentación de procesos y sistemas**: Al utilizar el enfoque de infraestructura como código, podemos reutilizar el código. Es importante que este esté documentado adecuadamente para que otros usuarios comprendan el propósito y funcionamiento del módulo.
- **Versionar todo**: Esto nos permite rastrear los cambios realizados. Si se comete un error, podemos retroceder a una versión estable.
- **Preferir cambios pequeños**: Realizar cambios pequeños para evitar grandes impactos.
- **Mantener los servicios continuamente disponibles**: Garantizar la disponibilidad continua es clave en la infraestructura.

### **Beneficios de la infraestructura como código**

- **Creación rápida y bajo demanda**: Con un único archivo de definición de infraestructura que almacena todas nuestras configuraciones, podemos crear múltiples veces la infraestructura sin necesidad de rehacer todo desde el principio.
- **Automatización**: Una vez creado el archivo de definición, podemos usar herramientas de **continuous integration** para automatizar la infraestructura.
- **Visibilidad y trazabilidad**: El versionamiento de la infraestructura como código permite una mayor visibilidad y trazabilidad, ya que todos los cambios quedan registrados.
- **Ambientes homogéneos**: Podemos crear varios ambientes a partir del mismo archivo de definición, cambiando únicamente algunos parámetros.

---

### **Mejores prácticas**

- **Modularidad**: Es recomendable dividir la infraestructura en módulos reutilizables para facilitar su mantenimiento y escalabilidad.
- **Mantener las configuraciones centralizadas**: Utilizar variables y archivos de configuración para gestionar parámetros y evitar valores "hardcoded".
- **Manejo seguro del estado**: Almacenar el archivo `terraform.tfstate` de manera remota (por ejemplo, en un bucket S3 con bloqueo de versión) para evitar problemas en equipos distribuidos.
- **Revisiones de código y pull requests**: Antes de aplicar cambios importantes en la infraestructura, hacer revisiones mediante pull requests para asegurar que los cambios han sido revisados por otros.

### **Ambientes**

Terraform permite la creación de múltiples ambientes (dev, stage, prod) con diferentes configuraciones. Puedes gestionar estos ambientes utilizando archivos `.tfvars` específicos para cada entorno.

- **Ambiente de desarrollo (dev)**: Se recomienda utilizar recursos más pequeños y económicos en este ambiente para reducir costos.
- **Ambiente de producción (prod)**: Aquí es importante configurar instancias y recursos con redundancia y alta disponibilidad.
  
Ejemplo de estructura para gestionar ambientes:

```bash
├── main.tf
├── variables.tf
├── dev.tfvars
├── prod.tfvars
```

Al aplicar los cambios para un ambiente en específico, puedes ejecutar:

```bash
terraform apply --var-file="dev.tfvars"
```

### **Automatización con CI/CD**

Integrar Terraform en un flujo de CI/CD es una excelente práctica para automatizar la gestión de la infraestructura. Puedes utilizar herramientas como Jenkins, GitLab CI, o GitHub Actions para automatizar el proceso de despliegue y validación.

Ejemplo de un pipeline básico en GitLab CI:

```yaml
stages:
  - validate
  - plan
  - apply

validate:
  script:
    - terraform init
    - terraform validate

plan:
  script:
    - terraform plan

apply:
  script:
    - terraform apply --auto-approve
```

Este pipeline primero inicializa el entorno, luego valida la configuración, y finalmente aplica los cambios automáticamente.

### **Seguridad**

- **Manejo seguro de credenciales**: Nunca almacenar credenciales en el código fuente. Utilizar herramientas como **AWS Secrets Manager** o **HashiCorp Vault** para gestionar los secretos de manera segura.
- **Control de acceso basado en roles (IAM)**: Asignar roles y permisos específicos a los recursos de Terraform mediante políticas de IAM para restringir el acceso según sea necesario.
- **Cifrado de datos**: Utilizar cifrado en reposo y en tránsito para proteger los datos sensibles, como el uso de **KMS (Key Management Service)** de AWS.
- **Seguridad en el estado**: Si almacenas el archivo `terraform.tfstate` en un bucket S3, asegúrate de habilitar el cifrado y el control de versiones para evitar modificaciones no autorizadas.

---

### **Manejo de variables en Terraform**

Para hacer escalable y reutilizable el archivo de definición de infraestructura, se recomienda no usar valores "hardcoded". Terraform permite crear variables de los siguientes tipos:

- **string**
- **number**
- **boolean**
- **map**
- **list**

Si no se declara un tipo, el valor por defecto será `string`. Sin embargo, es una buena práctica especificar el tipo de la variable.

Ejemplo de definición de variables:

```terraform
variable "ami_id" {
  type        = string
  description = "ID de la AMI"
}

variable "instance_type" {
  type        = string
  description = "Tipo de instancia"
}

variable "tags" {
  type        = map
  description = "Etiquetas para la instancia"
}
```

### **Asignar valores a las variables**

Los valores de las variables se pueden asignar de tres maneras:

1. Utilizando variables de entorno.
2. Pasándolos como argumentos en la línea de comandos.
3. Mediante un archivo `.tfvars` con formato `key = value`.

Ejemplo de archivo `.tfvars`:

```terraform
ami_id        = "ami-0ca0c67309196175e"
instance_type = "t2.micro"
tags = {
  Name       = "devops-tf"
  Environment = "Dev"
}
```

Para usar este archivo con variables:

```bash
terraform apply --var-file="dev.tfvars"
```

### **Destruir la infraestructura**

Para eliminar la infraestructura creada, se puede utilizar:

```bash
terraform destroy --var-file="dev.tfvars" -auto-approve
```











Nice — I went all-in. Below is a polished, step-by-step **README.md** in English that documents exactly what you did, the errors you hit, how you solved them, and the working link. Copy this into your repo as `README.md`.

````markdown
# Azure Function Terraform Lab — Step-by-Step

**Author:** Sergio Florez (@cheo-kt)  
**Repo:** `azfunction-tf`  
**Deployed Function App:** https://azfuncsergio06302005.azurewebsites.net

---

## Overview

This repo contains a small Terraform configuration that provisions an Azure Function App and the required resources:

- Resource Group  
- Storage Account  
- App Service Plan  
- Windows Function App

This README describes the step-by-step process I followed, the errors encountered, and how I fixed them.

---

## Prerequisites

- Azure CLI logged in (`az login`) and the correct subscription selected.  
- Terraform installed (tested with `terraform init`).  
- GitHub account and a Personal Access Token (PAT) for `git push` over HTTPS.

---

## Step-by-Step (what I actually ran)

1. Clone the repo and go into the folder:
    ```bash
    git clone <repo-url>
    cd azfunction-tf
    ```

2. Initialize Terraform:
    ```bash
    terraform init
    ```

3. Format and validate (optional but recommended):
    ```bash
    terraform fmt
    terraform validate
    ```

4. Run plan (Terraform asked for variable input):
    ```bash
    terraform plan
    # When prompted:
    # var.name_function
    # Enter a value: azfunc-sergio  <-- (original input caused a later error)
    ```

5. Fix variable values (see Errors section) and re-run plan:
    ```bash
    terraform plan
    # I used a safe final value: azfuncsergio06302005
    ```

6. Apply:
    ```bash
    terraform apply
    # Enter: yes
    ```

7. Verify with Azure CLI:
    ```bash
    az functionapp list --resource-group azfuncsergio06302005 -o table
    # Output confirmed: Function App is Running and DefaultHostName = azfuncsergio06302005.azurewebsites.net
    ```

8. Prepare Git commit and push:
    ```bash
    git config --global user.name "Sergio Florez"
    git config --global user.email "your_email@example.com"

    git add .
    git commit -m "terraform lab"
    # For pushing over HTTPS, use GitHub username + PAT as password:
    git push origin main
    ```

---

## Errors encountered and how I fixed them (detailed)

### 1) `name ("azfunc-sergio") can only consist of lowercase letters and numbers...`
**Symptom (Terraform error):**
````

Error: name ("azfunc-sergio") can only consist of lowercase letters and numbers, and must be between 3 and 24 characters long

```

**Cause:** I used `azfunc-sergio` (with a hyphen) as the Storage Account name. Storage account names:
- must be lowercase letters and numbers only (no `-`),  
- length 3–24,  
- globally unique across Azure.

**Fix:** Use a storage-account-safe name, e.g. `azfuncsergio06302005`. I changed the `azurerm_storage_account` `name` to a unique string (or use a distinct variable specifically for storage).

---

### 2) `RequestDisallowedByAzure` (region policy)
**Symptom (403 Forbidden on create):**
```

RequestDisallowedByAzure: Resource 'azfuncsergio' was disallowed by Azure: This policy maintains a set of best available regions...

```

**Cause:** My subscription (Azure for Students) has a policy restricting which regions are allowed. I had `westeurope` chosen.

**Fix:** Change the `location` variable to a permitted region (I used `eastus`):
- Edit `terraform.tfvars` or `variables.tf` default, or pass `-var="location=eastus"` when running `terraform plan/apply`.

---

### 3) `Invalid reference` when setting resource name
**Symptom:**
```

Error: A reference to a resource type must be followed by at least one attribute access...

````

**Cause:** I wrote `name = azfuncsergio` in `main.tf` (Terraform tried to parse that as a resource reference).

**Fix:** Wrap literal names in quotes:
```hcl
name = "azfuncsergio06302005"
````

---

### 4) `404 Not Found` & Terraform state inconsistency for Storage Account

**Symptom:**

```
ResourceNotFound: The Resource 'Microsoft.Storage/storageAccounts/azfuncsergio' under resource group 'azfuncsergio' was not found.
...
Error: Missing Resource Identity After Create
```

**Cause:** Terraform state referenced a storage account that either failed to create or was partially created and not present in Azure.

**Fix options:**

* **Easiest:** change the storage account name to a new unique name so Azure can create it cleanly.
* **Alternative (state cleanup):** remove the broken resource from the Terraform state so Terraform will recreate it:

  ```bash
  terraform state rm azurerm_storage_account.sa
  ```

  Then `terraform apply` to create the corrected resource.

---

### 5) Resource Group stuck deleting because contains resources

**Symptom while destroying:**

```
Error: deleting Resource Group "azfuncsergio": the Resource Group still contains Resources.
```

**Cause:** Azure was still showing a storage account inside the RG while Terraform tried to destroy it.

**Fix options:**

* **Manual:** go to Azure Portal → Resource Groups → open `azfuncsergio` → delete the lingering Storage Account, then let Terraform continue or re-run `terraform apply`.
* **Terraform provider flag (use with caution):** allow Terraform to delete RG even if it contains resources:

  ```hcl
  provider "azurerm" {
    features {
      resource_group {
        prevent_deletion_if_contains_resources = false
      }
    }
  }
  ```

  ⚠️ This will cause Terraform to call Azure API to delete all nested resources — be careful and only use it if you understand the consequences.

---

### 6) Long provisioning time / apply seems stuck

**Symptom:** `azurerm_windows_function_app.wfa: Still creating...` for > 10–20 minutes.

**Notes & fixes:**

* Windows Function Apps and App Service Plans can take a long time (10–15 minutes is not uncommon).
* Terraform sometimes waits for the final data-plane confirmation even after the app is listed as `Running`.
* Check the resource directly:

  ```bash
  az functionapp list --resource-group <rg-name> -o table
  ```

  If the app shows `Running` and has a `DefaultHostName`, you can:

  * Cancel the stuck `apply` with `CTRL+C`, then run:

    ```bash
    terraform refresh
    ```

    to sync state, or run `terraform apply` again.
* For faster tests in the future consider Linux Function Apps (they often provision faster).

---

### 7) Git push authentication error

**Symptom:**

```
remote: {"auth_status":"auth_error","body":"Invalid username or token. Password authentication is not supported for Git operations."}
fatal: Authentication failed ...
```

**Cause:** GitHub no longer accepts account passwords for HTTPS pushes.

**Fix:** Create a GitHub Personal Access Token (PAT) and use it as the password. Steps:

1. GitHub → Settings → Developer settings → Personal access tokens → Generate new token (classic).
2. Give `repo` scope, copy token.
3. Use your GitHub username and paste the token when `git push` asks for password.

Optionally configure `git credential.helper store` to avoid retyping.

---

## Verification (what I saw)

After the successful apply I ran:

```bash
az functionapp list --resource-group azfuncsergio06302005 -o table
```

Result snippet (confirmation):

```
Name                  Location    State    ResourceGroup         DefaultHostName                         AppServicePlan
azfuncsergio06302005  East US     Running  azfuncsergio06302005  azfuncsergio06302005.azurewebsites.net  azfuncsergio06302005
```

Open the site:

* [https://azfuncsergio06302005.azurewebsites.net](https://azfuncsergio06302005.azurewebsites.net) → shows the default Function App page (means the app is up).

---

## Files to commit (and what NOT to commit)

**Commit these:**

* `main.tf`
* `variables.tf`
* `outputs.tf`
* `README.md`
* any modules or examples you created


To remove state files already staged/pushed:

```bash
git rm --cached terraform.tfstate terraform.tfstate.backup
echo -e "terraform.tfstate\nterraform.tfstate.backup\n.terraform\n" >> .gitignore
git add .gitignore
git commit -m "ignore terraform state files"
git push origin main
```

---

## Quick tips & final notes

* Storage account names must be globally unique and only lowercase letters & numbers (3–24 chars).
* Use a `terraform.tfvars` file or `-var` flags to avoid interactive prompts:

  ```hcl
  # terraform.tfvars
  name_function = "azfuncsergio06302005"
  location = "eastus"
  ```
* If a `terraform apply` seems stuck but the resource shows `Running` in Azure, safe flow: `CTRL+C` → `terraform refresh`.
* Keep your PAT secret — do not hardcode it into repos.


<img width="1872" height="1024" alt="image" src="https://github.com/user-attachments/assets/522ad5ca-59b8-4ed2-9332-328d9974d124" />


```
```
