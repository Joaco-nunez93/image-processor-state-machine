<div align="center">



# 🖼️ AWS Step Functions Image Processor Lab



<img src="https://compote.slate.com/images/2119ff95-86f2-4546-a8fd-7b70ec58c9c6.jpeg?crop=1560%2C1040%2Cx0%2Cy0&width=370" alt="AWS Serverless" width="600"/>



### Aplicación serverless de procesamiento de imágenes construida con **AWS SAM** que demuestra orquestación avanzada con **Step Functions**, **procesamiento paralelo**, **Lambda Layers**, y **reintentos automáticos**.



[![AWS SAM](https://img.shields.io/badge/AWS-SAM-orange?style=for-the-badge&logo=amazonaws)](https://aws.amazon.com/serverless/sam/)
[![Step Functions](https://img.shields.io/badge/AWS-Step_Functions-FF4F8B?style=for-the-badge&logo=amazonaws)](https://aws.amazon.com/step-functions/)
[![Lambda](https://img.shields.io/badge/AWS-Lambda-orange?style=for-the-badge&logo=awslambda)](https://aws.amazon.com/lambda/)
[![Node.js](https://img.shields.io/badge/Node.js-22.x-339933?style=for-the-badge&logo=nodedotjs)](https://nodejs.org/)
[![S3](https://img.shields.io/badge/Amazon-S3-569A31?style=for-the-badge&logo=amazons3)](https://aws.amazon.com/s3/)
[![DynamoDB](https://img.shields.io/badge/Amazon-DynamoDB-4053D6?style=for-the-badge&logo=amazondynamodb)](https://aws.amazon.com/dynamodb/)



</div>



---



## 📋 Overview



Este repositorio contiene un laboratorio práctico que demuestra **orquestación de workflows serverless** con AWS Step Functions. El proyecto implementa un pipeline de procesamiento de imágenes que detecta, redimensiona, y almacena imágenes JPEG automáticamente cuando se suben a un bucket S3.



**Flujo de trabajo:**

1. Usuario sube una imagen al bucket S3 `source`
2. Evento S3 dispara la función Lambda `invokeImageProcessor`
3. Lambda inicia la ejecución del Step Functions State Machine
4. State Machine determina el tipo de archivo:
   - **Si es JPEG**: Ejecuta procesamiento paralelo (copia original + redimensiona thumbnail)
   - **Si NO es JPEG**: Elimina el archivo del bucket source
5. Se guardan los metadatos en DynamoDB (imagen original + thumbnail)
6. Se elimina la imagen original del bucket source (cleanup)
7. Todos los pasos quedan registrados en CloudWatch Logs con formato JSON



**Características principales:**

- ✅ **Step Functions Express**: Orquestación de bajo costo y alta velocidad
- ✅ **Procesamiento Paralelo**: Copia y redimensionado simultáneos
- ✅ **Lambda Layers**: Sharp para procesamiento de imágenes optimizado
- ✅ **Reintentos Automáticos**: Backoff exponencial configurado en cada paso
- ✅ **Validación de Tipo**: Solo procesa archivos JPEG válidos
- ✅ **Logs Estructurados**: JSON con `requestId` para trazabilidad completa
- ✅ **DynamoDB**: Persistencia de metadatos de imágenes procesadas
- ✅ **Infrastructure as Code**: Infraestructura completamente definida con SAM



## 🏗️ Arquitectura & Tecnologías



### **Core Technologies**



- **AWS SAM CLI** - Framework de desarrollo serverless basado en CloudFormation
- **AWS Step Functions (Express)** - Orquestación de workflows de bajo costo
- **AWS Lambda** - 6 funciones serverless especializadas
- **Amazon S3** - Almacenamiento de imágenes (source y destination)
- **Amazon DynamoDB** - Persistencia de metadatos de imágenes
- **Lambda Layers** - Sharp library para procesamiento de imágenes
- **CloudWatch Logs** - Monitoreo con logs estructurados en JSON
- **Node.js 22.x** - Runtime moderno para Lambda
- **JavaScript (ESM)** - Lenguaje de desarrollo con ES Modules



### **AWS Services**



| Servicio | Propósito | Configuración Clave |
|----------|-----------|---------------------|
| **S3 Bucket (Source)** | Recepción de imágenes | Trigger para Lambda |
| **S3 Bucket (Destination)** | Almacenamiento de procesadas | Originales + thumbnails |
| **Step Functions** | Orquestación del workflow | Express, logging ALL |
| **Lambda (6 funciones)** | Procesamiento modular | 128-256 MB, 10s timeout |
| **DynamoDB** | Metadatos de imágenes | PAY_PER_REQUEST |
| **Lambda Layer** | Sharp image library | nodejs22.x compatible |
| **CloudWatch Logs** | Trazabilidad y debugging | Formato JSON |



### **Development Tools**



- **AWS SAM CLI** - Herramienta de desarrollo local y deployment
- **AWS CLI** - Gestión de recursos y monitoreo
- **Sharp** - Librería de procesamiento de imágenes de alto rendimiento
- **npm** - Gestor de paquetes para dependencias



## 📁 Estructura del Proyecto



```
lab-image-processor-state-machine/
│
├── functions/                       # Código de las funciones Lambda
│   ├── invokeImageProcessor/        # Trigger: S3 → Step Functions
│   │   └── app.mjs
│   ├── getFileType/                 # Extrae extensión del archivo
│   │   └── app.mjs
│   ├── copyFile/                    # Copia archivo a bucket destino
│   │   └── app.mjs
│   ├── resizeImage/                 # Redimensiona imagen (Sharp)
│   │   └── app.mjs
│   ├── writeToDynamoDB/             # Guarda metadatos en DynamoDB
│   │   └── app.mjs
│   └── deleteFile/                  # Elimina archivo del bucket source
│       └── app.mjs
│
├── layers/                          # Lambda Layers
│   └── nodejs-sharp-layer/          # Sharp library para imágenes
│       ├── layer_content.zip        # Layer empaquetado
│       ├── nodejs/                  # Estructura de la layer
│       ├── package.json
│       └── package-lock.json
│
├── statemachine/                    # Definición de Step Functions
│   └── image-processor.asl.yaml     # ASL (Amazon States Language)
│
├── template.yaml                    # Plantilla SAM (infraestructura)
├── samconfig.toml                   # Configuración de despliegue SAM
└── README.md                        # Este archivo
```



### **Separación de Responsabilidades**



| Directorio | Propósito | Se despliega a AWS |
|------------|-----------|-------------------|
| `functions/` | Código fuente de Lambdas | ✅ Sí |
| `layers/` | Sharp library | ✅ Sí (como Layer) |
| `statemachine/` | Definición del workflow | ✅ Sí (como State Machine) |
| `template.yaml` | Infraestructura | ✅ Sí (como CloudFormation) |



## ✨ Componentes Clave



### **1️⃣ State Machine** (`image-processor.asl.yaml`)



El corazón del proyecto es la máquina de estados que orquesta todo el flujo de procesamiento.



```yaml
StartAt: GetFileType
States:
  GetFileType:        # Determina el tipo de archivo
  CheckFileType:      # Choice: ¿Es JPEG?
  ProcessFile:        # Parallel: Copia + Resize simultáneos
  WriteToDynamoDB:    # Persiste metadatos
  DeleteSourceFile:   # Cleanup del bucket source
  QuitMain:           # Estado de fallo general
```



**Diagrama del Flujo:**

```
                    ┌─────────────────┐
                    │   GetFileType   │
                    │   (Lambda)      │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  CheckFileType  │
                    │    (Choice)     │
                    └────────┬────────┘
                             │
           ┌─────────────────┼─────────────────┐
           │ jpeg            │                 │ other
           ▼                 │                 ▼
    ┌──────────────┐         │         ┌──────────────┐
    │ ProcessFile  │         │         │DeleteSource  │
    │  (Parallel)  │         │         │   File       │
    └──────┬───────┘         │         └──────────────┘
           │                 │                 │
    ┌──────┴──────┐          │                 │
    │             │          │                 │
    ▼             ▼          │                 │
┌───────┐   ┌─────────┐      │                 │
│ Copy  │   │ Resize  │      │                 │
│ File  │   │ Image   │      │                 │
└───┬───┘   └────┬────┘      │                 │
    │            │           │                 │
    └──────┬─────┘           │                 │
           │                 │                 │
    ┌──────▼──────┐          │                 │
    │WriteToDynamoDB          │                 │
    │  (Lambda)   │          │                 │
    └──────┬──────┘          │                 │
           │                 │                 │
    ┌──────▼──────┐          │                 │
    │DeleteSource │          │                 │
    │   File      │◄─────────┘                 │
    └──────┬──────┘                            │
           │                                   │
           ▼                                   ▼
        [END]                               [END]
```



---



### **2️⃣ Invoke Image Processor** (`invokeImageProcessor`)



Función trigger que recibe eventos S3 e inicia la ejecución del Step Functions.



```javascript
import { SFNClient, StartExecutionCommand } from "@aws-sdk/client-sfn";

export const handler = async (event, context) => {
    const executions = event.Records.map(async (record) => {
        const params = {
            stateMachineArn: process.env.STATE_MACHINE_ARN,
            input: JSON.stringify({ lambdaEvent: record })
        };
        return await stepFunctions.send(new StartExecutionCommand(params));
    });
    return await Promise.all(executions);
};
```



**Características:**
- ✅ Procesa múltiples archivos en paralelo
- ✅ Pasa el evento S3 completo al State Machine
- ✅ Logging estructurado con `requestId`



---



### **3️⃣ Get File Type** (`getFileType`)



Extrae la extensión del archivo para determinar si es procesable.



```javascript
export const handler = async (event, context) => {
    const filename = event.s3.object.key;
    const index = filename.lastIndexOf('.');
    const fileType = index > 0 ? filename.substring(index + 1) : null;
    return fileType;
};
```



**Retorna:** `"jpeg"`, `"png"`, `"gif"`, `null`, etc.



---



### **4️⃣ Copy File** (`copyFile`)



Copia la imagen original al bucket de destino.



```javascript
import { S3Client, CopyObjectCommand } from "@aws-sdk/client-s3";

export const handler = async (event, context) => {
    const params = {
        Bucket: process.env.DESTINATION_BUCKET,
        CopySource: encodeURI(`/${sourceBucket}/${key}`),
        Key: key
    };
    await s3.send(new CopyObjectCommand(params));
    return { region, bucket: destinationBucket, key };
};
```



---



### **5️⃣ Resize Image** (`resizeImage`)



Redimensiona la imagen a 150px de ancho usando Sharp.



```javascript
import sharp from "sharp";

export const handler = async (event, context) => {
    const { Body } = await s3.send(new GetObjectCommand({ Bucket, Key }));
    const resizedImage = await sharp(await Body.transformToByteArray())
        .resize(150)
        .toBuffer();
    const newKey = key.replace(".jpeg", "-small.jpeg");
    await s3.send(new PutObjectCommand({ Bucket, Key: newKey, Body: resizedImage }));
    return { region, bucket, key: newKey };
};
```



**Características:**
- ✅ Usa Lambda Layer con Sharp precompilado
- ✅ Memoria aumentada a 256 MB para procesamiento
- ✅ Genera thumbnails con sufijo `-small`



---



### **6️⃣ Write to DynamoDB** (`writeToDynamoDB`)



Persiste los metadatos de las imágenes procesadas.



```javascript
import { DynamoDBClient, PutItemCommand } from "@aws-sdk/client-dynamodb";

export const handler = async (event, context) => {
    const item = {
        TableName: "thumbnails",
        Item: {
            original: { S: `${region}|${bucket}|${key}` },
            thumbnail: { S: `${region}|${bucket}|${thumbnailKey}` },
            timestamp: { N: `${Date.now()}` }
        }
    };
    await dynamoDB.send(new PutItemCommand(item));
    return true;
};
```



**Esquema DynamoDB:**

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `original` | String (PK) | `region\|bucket\|key` de la imagen original |
| `thumbnail` | String | `region\|bucket\|key` del thumbnail |
| `timestamp` | Number | Timestamp de procesamiento |



---



### **7️⃣ Delete File** (`deleteFile`)



Elimina archivos del bucket source (cleanup o archivos no válidos).



```javascript
import { S3Client, DeleteObjectCommand } from "@aws-sdk/client-s3";

export const handler = async (event, context) => {
    const params = {
        Bucket: event.s3.bucket.name,
        Key: event.s3.object.key
    };
    await s3.send(new DeleteObjectCommand(params));
    return true;
};
```



---



### **8️⃣ Sharp Lambda Layer**



Layer precompilada con la librería Sharp para procesamiento de imágenes de alto rendimiento.



```yaml
SharpLayer:
  Type: AWS::Serverless::LayerVersion
  Properties:
    LayerName: nodejs-sharp-layer
    ContentUri: layers/nodejs-sharp-layer/layer_content.zip
    CompatibleRuntimes:
      - nodejs22.x
    CompatibleArchitectures:
      - x86_64
      - arm64
```



**Ventajas de usar Layer:**
- ✅ Reduce el tamaño del deployment de Lambda
- ✅ Reutilizable entre múltiples funciones
- ✅ Sharp precompilado para arquitectura Lambda
- ✅ Actualizable independientemente del código



## ☁️ Recursos AWS Creados



Al ejecutar `sam deploy`, se crean los siguientes recursos:



| Recurso | Tipo AWS | Propósito | Costo Estimado |
|---------|----------|-----------|----------------|
| **State Machine** | `AWS::StepFunctions::StateMachine` | Orquestación del workflow | ~$0.025/1000 ejecuciones |
| **Lambda (x6)** | `AWS::Lambda::Function` | Procesamiento modular | Gratis (1M invocaciones/mes) |
| **Lambda Layer** | `AWS::Lambda::LayerVersion` | Sharp library | Gratis |
| **S3 Bucket (Source)** | `AWS::S3::Bucket` | Recepción de imágenes | ~$0.023/GB almacenado |
| **S3 Bucket (Destination)** | `AWS::S3::Bucket` | Imágenes procesadas | ~$0.023/GB almacenado |
| **DynamoDB Table** | `AWS::DynamoDB::Table` | Metadatos de imágenes | PAY_PER_REQUEST |
| **CloudWatch Log Group** | `AWS::Logs::LogGroup` | Logs del State Machine | $0.50/GB almacenado |
| **IAM Roles** | `AWS::IAM::Role` | Permisos de ejecución | Gratis |



**💰 Costo Total Estimado:**
- **Free Tier**: Completamente gratis (dentro de límites)
- **Post Free Tier**: ~$0.50-$2.00/mes con uso moderado (100-500 imágenes/día)



## 🔄 Flujo de Funcionamiento



### **Escenario 1: Imagen JPEG Válida**



```
┌──────────┐   Upload    ┌─────────────┐   Trigger    ┌────────────────┐
│          │   .jpeg     │             │   Lambda     │                │
│  Usuario │ ──────────▶ │ S3 (Source) │ ───────────▶ │ invokeImage    │
│          │             │             │              │ Processor      │
└──────────┘             └─────────────┘              └───────┬────────┘
                                                              │
                                              StartExecution  │
                                                              ▼
                                                    ┌─────────────────┐
                                                    │ Step Functions  │
                                                    │ State Machine   │
                                                    └────────┬────────┘
                                                             │
                              ┌───────────────────────────────┤
                              │                               │
                              ▼                               ▼
                    ┌─────────────────┐             ┌─────────────────┐
                    │   Copy File     │             │  Resize Image   │
                    │   (Parallel)    │             │   (Parallel)    │
                    └────────┬────────┘             └────────┬────────┘
                              │                               │
                              └───────────┬───────────────────┘
                                          │
                                          ▼
                              ┌─────────────────────┐
                              │  WriteToDynamoDB    │
                              │  (Metadata)         │
                              └──────────┬──────────┘
                                         │
                                         ▼
                              ┌─────────────────────┐
                              │  DeleteSourceFile   │
                              │  (Cleanup)          │
                              └──────────┬──────────┘
                                         │
                                         ▼
                                     [SUCCESS]
```



**Resultado Final:**
- ✅ Imagen original en `s3://destination-bucket/imagen.jpeg`
- ✅ Thumbnail en `s3://destination-bucket/imagen-small.jpeg`
- ✅ Registro en DynamoDB con metadatos
- ✅ Imagen eliminada del bucket source



---



### **Escenario 2: Archivo No JPEG**



```
┌──────────┐   Upload    ┌─────────────┐   Trigger    ┌────────────────┐
│          │   .png      │             │   Lambda     │                │
│  Usuario │ ──────────▶ │ S3 (Source) │ ───────────▶ │ invokeImage    │
│          │             │             │              │ Processor      │
└──────────┘             └─────────────┘              └───────┬────────┘
                                                              │
                                                              ▼
                                                    ┌─────────────────┐
                                                    │  GetFileType    │
                                                    │  returns: "png" │
                                                    └────────┬────────┘
                                                             │
                                                             ▼
                                                    ┌─────────────────┐
                                                    │  CheckFileType  │
                                                    │  ≠ "jpeg"       │
                                                    └────────┬────────┘
                                                             │
                                                             ▼
                                                    ┌─────────────────┐
                                                    │ DeleteSourceFile│
                                                    │ (Reject file)   │
                                                    └────────┬────────┘
                                                             │
                                                             ▼
                                                         [END]
```



**Resultado Final:**
- ❌ Archivo eliminado del bucket source
- ❌ No se crea copia en destination
- ❌ No se registra en DynamoDB



## 🚀 Comandos Útiles



### **Instalación Inicial**



```bash
# Verificar que SAM CLI esté instalado
sam --version

# Verificar que AWS CLI esté configurado
aws sts get-caller-identity
```



### **Validación de Template**



```bash
# Validar sintaxis de template.yaml
sam validate

# Validar con linting estricto
sam validate --lint
```



### **Build & Deploy**



```bash
# Construir la aplicación (genera .aws-sam/build/)
sam build

# Primera vez: deployment guiado
sam deploy --guided

# Deployments subsiguientes (usa samconfig.toml)
sam build && sam deploy

# Deployment sin confirmación (CI/CD)
sam build && sam deploy --no-confirm-changeset
```



**Durante `sam deploy --guided`:**
```
Stack Name [sam-app]: lab-image-processor-state-machine
AWS Region [us-east-1]: us-east-2
Confirm changes before deploy [y/N]: y
Allow SAM CLI IAM role creation [Y/n]: Y
Save arguments to samconfig.toml [Y/n]: Y
```



### **Testing & Monitoring**



#### **1. Obtener nombres de los buckets**



**Bash:**
```bash
SOURCE_BUCKET=$(aws cloudformation describe-stacks \
  --stack-name lab-image-processor-state-machine \
  --query "Stacks[0].Outputs[?OutputKey=='SourceS3Bucket'].OutputValue" \
  --output text)

echo "Source Bucket: $SOURCE_BUCKET"
```



**PowerShell:**
```powershell
$SOURCE_BUCKET = aws s3api list-buckets `
  --query "Buckets[?contains(Name, 'source-bucket')].Name" `
  --output text

Write-Output "Source Bucket: $SOURCE_BUCKET"
```



#### **2. Subir una imagen de prueba**



**Bash:**
```bash
# Subir imagen JPEG (será procesada)
aws s3 cp test-image.jpeg s3://$SOURCE_BUCKET/

# Subir archivo no JPEG (será eliminado)
aws s3 cp test-file.png s3://$SOURCE_BUCKET/
```



**PowerShell:**
```powershell
# Subir imagen JPEG (será procesada)
aws s3 cp test-image.jpeg s3://$SOURCE_BUCKET/

# Subir archivo no JPEG (será eliminado)
aws s3 cp test-file.png s3://$SOURCE_BUCKET/
```



#### **3. Monitorear ejecuciones del State Machine**



```bash
# Ver ejecuciones recientes
aws stepfunctions list-executions \
  --state-machine-arn <STATE_MACHINE_ARN> \
  --max-results 10

# Ver detalles de una ejecución específica
aws stepfunctions describe-execution \
  --execution-arn <EXECUTION_ARN>
```



#### **4. Monitorear logs en tiempo real**



```bash
# Logs del State Machine
aws logs tail /aws/vendedlogs/ImageProcessorStateMachine-Logs --follow

# Logs de función específica
aws logs tail /aws/lambda/resizeImage --follow

# Filtrar solo errores
aws logs tail /aws/lambda/resizeImage --filter-pattern "error"
```



#### **5. Verificar imágenes procesadas**



```bash
# Listar imágenes en bucket destino
aws s3 ls s3://lab-s3-destination-bucket-<ACCOUNT_ID>/

# Descargar thumbnail generado
aws s3 cp s3://lab-s3-destination-bucket-<ACCOUNT_ID>/imagen-small.jpeg .
```



#### **6. Verificar registros en DynamoDB**



```bash
# Escanear tabla de thumbnails
aws dynamodb scan --table-name thumbnails

# Buscar por imagen específica
aws dynamodb get-item \
  --table-name thumbnails \
  --key '{"original": {"S": "us-east-2|bucket-name|imagen.jpeg"}}'
```



#### **7. CloudWatch Logs Insights**



```sql
-- Ver todas las ejecuciones
fields @timestamp, type, details.name, details.output
| filter type = "TaskStateExited"
| sort @timestamp desc
| limit 50

-- Ver errores
fields @timestamp, type, details.error, details.cause
| filter type = "TaskFailed" or type = "ExecutionFailed"
| sort @timestamp desc
```



### **Cleanup (Eliminar Stack)**



```bash
# Vaciar buckets S3 primero (requerido para eliminar)
aws s3 rm s3://lab-s3-source-bucket-<ACCOUNT_ID>/ --recursive
aws s3 rm s3://lab-s3-destination-bucket-<ACCOUNT_ID>/ --recursive

# Eliminar todos los recursos
sam delete --stack-name lab-image-processor-state-machine

# Sin confirmación
sam delete --no-prompts
```



> [!WARNING]
> `sam delete` eliminará permanentemente:
> - State Machine
> - Lambda Functions
> - Lambda Layer
> - S3 Buckets (deben estar vacíos)
> - DynamoDB Table
> - IAM Roles
> - CloudWatch Log Groups



## 💡 Ventajas del Proyecto



| Ventaja | Descripción |
|---------|-------------|
| **🔄 Orquestación Visual** | Step Functions proporciona visualización del workflow en tiempo real |
| **⚡ Procesamiento Paralelo** | Copia y redimensionado ejecutados simultáneamente |
| **🛡️ Reintentos Automáticos** | Backoff exponencial configurado en cada paso crítico |
| **📊 Observabilidad Completa** | Logs estructurados JSON + execution history en Step Functions |
| **🧩 Modularidad** | 6 funciones Lambda especializadas y reutilizables |
| **📦 Lambda Layers** | Sharp precompilado, reutilizable y actualizable |
| **💰 Bajo Costo** | Step Functions Express (~$0.025/1000 ejecuciones) |
| **🔒 Seguridad** | IAM roles con permisos mínimos por función |
| **📝 Infrastructure as Code** | Infraestructura reproducible con SAM |
| **🎯 Validación de Tipo** | Solo procesa archivos JPEG, rechaza otros |



## 📚 Casos de Uso



Este patrón arquitectónico (S3 → Lambda → Step Functions) es ideal para:



| Caso de Uso | Descripción | Aplicación |
|-------------|-------------|------------|
| 🖼️ **Image Processing** | Procesamiento de imágenes con múltiples pasos | Thumbnails, watermarks, filtros |
| 📹 **Video Transcoding** | Conversión de video con orquestación | VOD, streaming platforms |
| 📄 **Document Processing** | Extracción y transformación de documentos | OCR, PDF processing |
| 🔄 **ETL Pipelines** | Transformación de datos con pasos definidos | Data lakes, analytics |
| 📧 **Email Campaigns** | Procesamiento de campañas con múltiples pasos | Marketing automation |
| 🤖 **ML Inference** | Pipelines de inferencia con pre/post procesamiento | Image classification |
| 📦 **Order Processing** | Workflows de e-commerce con validaciones | Inventory, fulfillment |
| 🔐 **Compliance Workflows** | Procesos regulatorios con auditoría | KYC, document verification |



## 🧠 Profundizando en Step Functions



### **Express vs Standard Workflows**



| Característica | Express (Este Lab) | Standard |
|----------------|-------------------|----------|
| **Duración máxima** | 5 minutos | 1 año |
| **Precio** | ~$0.025/1000 ejecuciones | ~$0.025/transición |
| **Ejecución** | At-least-once | Exactly-once |
| **Casos de uso** | Alto volumen, corta duración | Larga duración, workflows humanos |
| **Historial** | CloudWatch Logs | Console + API |



### **Configuración de Reintentos**



```yaml
Retry:
  - ErrorEquals: ["States.TaskFailed", "States.Timeout"]
    IntervalSeconds: 5      # Espera inicial
    MaxAttempts: 2          # Intentos adicionales
    BackoffRate: 2.0        # Multiplicador (5s → 10s → 20s)
  - ErrorEquals: ["States.ALL"]
    IntervalSeconds: 2
    MaxAttempts: 2
    BackoffRate: 2.0
```



**Timeline de Reintentos:**
1. **Intento 1**: Inmediato → Error
2. **Espera**: 5 segundos
3. **Intento 2**: Retry 1 → Error
4. **Espera**: 10 segundos (5 × 2.0)
5. **Intento 3**: Retry 2 → Error/Success
6. **Si falla**: Ejecuta `Catch` → Estado de fallo



### **Manejo de Errores con Catch**



```yaml
Catch:
  - ErrorEquals: ["States.ALL"]
    Next: QuitCopy       # Estado de fallo específico
```



**Estados de Fallo Definidos:**
- `QuitCopy`: Error en copia de archivo
- `QuitResize`: Error en redimensionado
- `QuitMain`: Error general del workflow



## 🔧 Configuración Avanzada



### **Ajustar Tamaño del Thumbnail**



En `functions/resizeImage/app.mjs`:

```javascript
// Cambiar de 150px a otro tamaño
const resizedImage = await sharp(await Body.transformToByteArray())
    .resize(300)  // Nuevo ancho
    .toBuffer();
```



### **Procesar Otros Formatos**



En `statemachine/image-processor.asl.yaml`:

```yaml
CheckFileType:
  Type: Choice
  Choices:
    - Variable: $.results.fileType.Payload
      StringEquals: "jpeg"
      Next: ProcessFile
    - Variable: $.results.fileType.Payload
      StringEquals: "png"      # Añadir PNG
      Next: ProcessFile
    - Variable: $.results.fileType.Payload
      StringEquals: "webp"     # Añadir WebP
      Next: ProcessFile
  Default: DeleteSourceFile
```



### **Añadir Notificaciones de Éxito/Error**



En `template.yaml`, añadir SNS Topic:

```yaml
ProcessingNotifications:
  Type: AWS::SNS::Topic
  Properties:
    TopicName: ImageProcessingNotifications
```



---



## 📋 Resumen del Lab



Este laboratorio cubre de forma práctica los siguientes conceptos:



### **🏗️ Infraestructura**

- **AWS SAM**: Creación y despliegue de aplicaciones serverless
- **Step Functions**: Orquestación visual de workflows
- **Lambda Layers**: Compartir dependencias entre funciones
- **IAM Roles**: Permisos granulares por función



### **🔄 Patrones de Diseño**

- **Parallel State**: Ejecución simultánea de tareas independientes
- **Choice State**: Branching condicional basado en datos
- **Retry/Catch**: Manejo robusto de errores con backoff
- **Event-Driven**: S3 como trigger del pipeline



### **📊 Observabilidad**

- **CloudWatch Logs**: Logging estructurado en JSON
- **Execution History**: Trazabilidad paso a paso en Step Functions
- **Metrics**: Métricas automáticas de ejecuciones



### **✅ Mejores Prácticas**

- **Modularidad**: Funciones pequeñas y especializadas
- **Idempotencia**: Operaciones seguras para reintentos
- **Least Privilege**: Permisos mínimos necesarios
- **Infrastructure as Code**: Todo versionado y reproducible



---



## 🛠️ Próximos Pasos Sugeridos



### **Nivel Básico**
- [ ] Añadir soporte para PNG y WebP
- [ ] Generar múltiples tamaños de thumbnail (small, medium, large)
- [ ] Añadir notificación SNS al completar procesamiento
- [ ] Implementar verificación de tamaño máximo de archivo



### **Nivel Intermedio**
- [ ] **Añadir watermark**: Sobreponer logo en imágenes
- [ ] **Face detection**: Usar Amazon Rekognition para detectar rostros
- [ ] **Moderación de contenido**: Filtrar imágenes inapropiadas
- [ ] **Procesamiento por lotes**: Agregar múltiples imágenes en una ejecución
- [ ] **Dashboard**: CloudWatch Dashboard con métricas del pipeline



### **Nivel Avanzado**
- [ ] **Multi-región**: Replicar imágenes a bucket en otra región
- [ ] **EventBridge**: Emitir eventos custom al procesar imágenes
- [ ] **API Gateway**: Exponer endpoint para subida directa
- [ ] **Callback Pattern**: Notificar a sistemas externos al completar
- [ ] **Cost Optimization**: Migrar a ARM64 (Graviton2)



---



## 📖 Recursos Adicionales



### **Documentación Oficial**
- [AWS Step Functions Developer Guide](https://docs.aws.amazon.com/step-functions/latest/dg/)
- [Amazon States Language](https://states-language.net/spec.html)
- [AWS SAM Documentation](https://docs.aws.amazon.com/serverless-application-model/)
- [Sharp Documentation](https://sharp.pixelplumbing.com/)
- [Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/chapter-layers.html)



### **Tutoriales y Workshops**
- [Step Functions Workshop](https://catalog.workshops.aws/stepfunctions/)
- [Serverless Image Handler](https://aws.amazon.com/solutions/implementations/serverless-image-handler/)
- [SAM Workshop](https://catalog.workshops.aws/complete-aws-sam/)



### **Best Practices**
- [Step Functions Best Practices](https://docs.aws.amazon.com/step-functions/latest/dg/sfn-best-practices.html)
- [Lambda Best Practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [Well-Architected Serverless Lens](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/)



---



## ⚠️ Troubleshooting



### **Problema: "Sharp module not found"**

**Causa**: Lambda Layer no incluida o incompatible.

**Solución**:
```bash
# Verificar que la layer esté incluida en resizeImage
sam build
sam deploy

# O reempaquetar la layer manualmente
cd layers/nodejs-sharp-layer
npm install
zip -r layer_content.zip nodejs/
```



### **Problema: Imágenes no se procesan**

**Causa**: El archivo no es JPEG o tiene extensión incorrecta.

**Solución**:
- Verificar que el archivo tenga extensión `.jpeg` (no `.jpg`)
- Revisar logs de `getFileType` para ver qué extensión se detecta



### **Problema: "Access Denied" al copiar archivos**

**Causa**: Permisos insuficientes en IAM Role.

**Solución**:
```bash
# Verificar que las políticas S3 estén aplicadas
aws iam list-attached-role-policies --role-name <ROLE_NAME>
```



### **Problema: State Machine falla con timeout**

**Causa**: Lambda tarda más de 3 segundos (configurado en ASL).

**Solución**:
```yaml
# En image-processor.asl.yaml, aumentar TimeoutSeconds
TimeoutSeconds: 10
```



---



## 📊 Métricas y Monitoreo



### **Métricas de Step Functions**

| Métrica | Descripción | Alarma Sugerida |
|---------|-------------|-----------------|
| `ExecutionsStarted` | Ejecuciones iniciadas | N/A |
| `ExecutionsSucceeded` | Ejecuciones exitosas | < 95% del total |
| `ExecutionsFailed` | Ejecuciones fallidas | > 5% del total |
| `ExecutionTime` | Duración de ejecución | > 30 segundos |



### **CloudWatch Dashboard Sugerido**

```bash
# Crear dashboard con métricas clave
aws cloudwatch put-dashboard \
  --dashboard-name "ImageProcessor" \
  --dashboard-body file://dashboard.json
```



---



## 📝 Licencia



Este proyecto es de código abierto y está disponible bajo la licencia MIT para fines educativos.



---



## 👤 Autor



**Joaquín**  
Proyecto de laboratorio para aprender orquestación serverless con AWS Step Functions



---



## 📮 Contacto



¿Preguntas o sugerencias? Abre un issue en el repositorio.



---



## 📚 Apéndice: Comandos Rápidos de Referencia



```bash
# 🔨 Build & Deploy
sam build                              # Construir aplicación
sam deploy                             # Desplegar (requiere samconfig.toml)
sam deploy --guided                    # Deployment guiado (primera vez)

# 📤 Testing (Upload to S3)
aws s3 cp image.jpeg s3://<SOURCE_BUCKET>/

# 📊 Logs & Monitoring
aws logs tail /aws/vendedlogs/ImageProcessorStateMachine-Logs --follow
aws stepfunctions list-executions --state-machine-arn <ARN>

# 🔍 Verificar Resultados
aws s3 ls s3://<DESTINATION_BUCKET>/
aws dynamodb scan --table-name thumbnails

# ✅ Validation
sam validate                           # Validar template.yaml
sam validate --lint                    # Validar con linting

# 🗑️ Cleanup
aws s3 rm s3://<SOURCE_BUCKET>/ --recursive
aws s3 rm s3://<DESTINATION_BUCKET>/ --recursive
sam delete                             # Eliminar stack y recursos
```



---



## 🎯 Checklist de Deployment



Antes de hacer `sam deploy --guided` por primera vez:

- [ ] AWS CLI configurado (`aws configure`)
- [ ] SAM CLI instalado y actualizado (`sam --version >= 1.100.0`)
- [ ] Template validado (`sam validate`)
- [ ] Región AWS decidida (ej: `us-east-2`)
- [ ] Stack name decidido (ej: `lab-image-processor-state-machine`)



Después del deployment:

- [ ] Imagen JPEG de prueba subida al bucket source
- [ ] Ejecución de Step Functions verificada en consola
- [ ] Imagen original copiada al bucket destino
- [ ] Thumbnail generado en bucket destino (`-small.jpeg`)
- [ ] Registro creado en DynamoDB
- [ ] Imagen eliminada del bucket source
- [ ] Logs verificados en CloudWatch



---



**⭐ Si este laboratorio te resultó útil para aprender Step Functions, considera darle una estrella!**



**¡Feliz aprendizaje! 🚀**

