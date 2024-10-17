# Instalación de PyTorch en CCAD y Ejemplo de Uso
Asumiendo que ya se tiene una cuenta de usuario en el CCAD y que Jupyter está instalado (si no, referirse a [Instrucciones de instalación de Jupyter en CCAD](https://github.com/jipphysics/jupyter-ccad)). 

## Conexión al Nodo 
Para conectarse, ejecutar el siguiente comando: 
```bash 
ssh -L 1234:localhost:1234 usuario@jupyter.ccad.unc.edu.ar
```
Es importante elegir un puerto para la conexión, por ejemplo: `1234`.
## Verificar Versión de ROCm
Para comprobar la versión de ROCm, ejecutar:
```bash 
[jcoviedo@jupyter ~]$ rocminfo
```
El resultado esperado debería ser algo como:
```bash 
ROCk module version 6.8.5 is loaded 
===================== 
HSA System Attributes 
===================== 
Runtime Version: 1.14 
Runtime Ext Version: 1.6 
System Timestamp Freq.: 1000.000000MHz 
...
```
Si no se obtiene una salida similar, es posible que no se tengan los permisos adecuados. En ese caso, contactar a soporte: soporte@ccad.unc.edu.ar.

## Activar el Entorno de Trabajo
Dependiendo del método de instalación de Jupyter, activar el entorno correspondiente. Si se usó `micromamba`, el comando sería:
```bash 
[jcoviedo@jupyter ~]$ micromamba activate jnb-env
```

## Instalación de PyTorch
Dirigirse a [pytorch.org](https://pytorch.org/get-started/locally/) y seleccionar las siguientes opciones:

-   Stable
-   Linux
-   Pip
-   Python
-   ROCm (alguna versión)

Copiar el código proporcionado y ejecutarlo, por ejemplo:
```bash 
(jnb-env) [jcoviedo@jupyter ~]$ pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm6.1
```
O bien:

```bash 
(jnb-env) [jcoviedo@jupyter ~]$ pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm6.1
```

## Abrir Jupyter
Una vez instalado PyTorch, abrir Jupyter con:
```bash 
(jnb-env) [jcoviedo@jupyter ~]$ jupyter notebook --no-browser --port=1234
```
El sistema generará un enlace como el siguiente:
```bash 
http://localhost:1235/tree?token=8a1520a7bd3aa8b22cb0218be18d5e86c2bcbf877b048587
```
Es importante utilizar el mismo puerto por el cual se conectaron. Si Jupyter no se abre y muestra un error de `connection_refused`, conectarse al nodo nuevamente con un puerto distinto e intentar de nuevo.

## Comprobación de Instalación
Crear un nuevo notebook y agregar el siguiente código:

```python
import torch print(f"number of GPUs:
torch.cuda.device_count()}")
print(torch.__version__)
```
Al ejecutarlo, debería mostrar algo como:
```typescript
number of GPUs: 1
2.4.0+rocm6.1
```

## Ver Carga de GPU

Para ver la carga actual de la GPU, ejecutar en la terminal conectada al nodo:
```bash 
nvtop
```
## Ejemplo con LLM

Crear un nuevo notebook y agregar el siguiente código:

```python
import transformers
import torch
```
Si faltan librerías, se pueden instalar ejecutando:
```bash 
pip install transformers
```
en la notebook o en una terminal conectada al nodo. Luego, reiniciar el kernel si es necesario.

En una nueva celda, agregar:
```python
model_id = "meta-llama/Meta-Llama-3.1-8B-Instruct" device = torch.device("cuda" if torch.cuda.is_available() else raise ValueError("No se reconoció GPU."))
pipeline = transformers.pipeline(
	"text-generation", model=model_id,
	model_kwargs={"torch_dtype": torch.bfloat16},
	device=device
)
```
Finalmente, en otra celda:
```python
messages = [
{"role": "system", "content": "Tú eres un cordobés, quiero que me respondas todo como cordobés"},
{"role": "user", "content": "¿Cuál es la mejor bebida de Argentina?"}
]
output = pipeline(messages, max_new_tokens=500)
print(output[0]["generated_text"][2]['content'])
```
