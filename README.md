# ComfyUI com NVIDIA no Docker Desktop (Windows 11)

Guia para instalar e executar o **ComfyUI** usando Docker Desktop no Windows 11, com aceleração pela GPU NVIDIA GeForce RTX 4050.

O procedimento utiliza o projeto [ComfyUI-Nvidia-Docker](https://github.com/mmartial/ComfyUI-Nvidia-Docker) e o WSL 2.

> **Objetivo:** executar o ComfyUI localmente e acessá-lo pelo navegador em `http://localhost:8188`.

---

## 1. Ambiente utilizado

- **Sistema operacional:** Windows 11
- **GPU:** NVIDIA GeForce RTX 4050
- **Docker Desktop:** versão 4.98
- **Backend Linux:** WSL 2 com Ubuntu
- **Projeto:** [mmartial/ComfyUI-Nvidia-Docker](https://github.com/mmartial/ComfyUI-Nvidia-Docker)

## 2. Preparar o Windows e instalar o WSL 2

1. Abra o menu Iniciar.
2. Digite `PowerShell`.
3. Clique com o botão direito em **Windows PowerShell** e selecione **Executar como administrador**.
4. Execute:

```powershell
wsl --install -d Ubuntu
```

5. Reinicie o computador caso o Windows solicite.
6. Depois de reiniciar, abra o aplicativo **Ubuntu** pelo menu Iniciar.
7. Na primeira execução, crie um nome de usuário e uma senha para o Linux.

> **Atenção:** ao digitar a senha no terminal Linux, os caracteres não aparecem na tela. Isso é normal.

## 3. Configurar o Docker Desktop

1. Abra o **Docker Desktop**.
2. Clique na engrenagem para abrir **Settings**.
3. Em **General**, habilite:

   - `Use the WSL 2 based engine`

4. Entre em **Resources → WSL Integration**.
5. Habilite:

   - `Enable integration with my default WSL distro`
   - A integração da distribuição `Ubuntu`

6. Clique em **Apply & Restart**.

## 4. Atualizar o WSL e testar a GPU NVIDIA

Antes de instalar o ComfyUI, vamos confirmar que o Docker consegue acessar a placa de vídeo.

Abra o **PowerShell** normalmente (não precisa ser como administrador) e execute, um comando por vez:

```powershell
wsl --update
```

```powershell
wsl --shutdown
```

Abra novamente o Docker Desktop e aguarde ele terminar a inicialização.

Em seguida, no PowerShell, execute o teste da GPU:

```powershell
docker run --rm -it --gpus=all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark
```

O Docker poderá baixar uma imagem de teste. Aguarde a execução.

**Resultado esperado:** a saída deve identificar uma GPU NVIDIA, com um nome semelhante a:

```text
GPU Device 0: "GeForce RTX 4050 ..."
```

Se a GPU for identificada, o acesso à NVIDIA pelo Docker está funcionando e podemos continuar.

> Se o teste apresentar erro, não prossiga para a instalação ainda. Confira se o driver NVIDIA do Windows está atualizado, se o WSL 2 está funcionando e se a integração do Ubuntu está habilitada no Docker Desktop.

## 5. Baixar o projeto ComfyUI-Nvidia-Docker

Abra o aplicativo **Ubuntu**.

Execute os comandos abaixo, um por vez.

### 5.1. Instalar o Git

```bash
sudo apt update
sudo apt install -y git
```

### 5.2. Baixar o repositório

```bash
git clone https://github.com/mmartial/ComfyUI-Nvidia-Docker.git
```

### 5.3. Entrar na pasta do projeto

```bash
cd ComfyUI-Nvidia-Docker
```

### 5.4. Criar as pastas de trabalho

```bash
mkdir -p run basedir
```

## 6. Ajustar o arquivo de configuração para WSL 2

Ainda no Ubuntu, execute:

```bash
notepad.exe compose.yaml
```

O arquivo `compose.yaml` será aberto no Bloco de Notas do Windows.

Faça as duas alterações abaixo.

### 6.1. Habilitar o volume do WSL

Localize esta linha:

```yaml
# - /usr/lib/wsl:/usr/lib/wsl # Required for Windows Subsystem for Linux 2
```

Retire o `#` do início, deixando assim:

```yaml
- /usr/lib/wsl:/usr/lib/wsl # Required for Windows Subsystem for Linux 2
```

### 6.2. Habilitar o caminho das bibliotecas NVIDIA

Localize:

```yaml
# - LD_LIBRARY_PATH=/usr/lib/wsl/lib
```

Retire o `#` do início, deixando assim:

```yaml
- LD_LIBRARY_PATH=/usr/lib/wsl/lib
```

Salve o arquivo com **Ctrl + S** e feche o Bloco de Notas.

> **Importante:** mantenha a indentação YAML original. Não altere os espaços no começo das linhas; apenas remova o comentário indicado.

## 7. Iniciar o ComfyUI

Volte ao terminal Ubuntu. Confirme que ele ainda está na pasta `ComfyUI-Nvidia-Docker`.

Execute:

```bash
docker compose up -d
```

Na primeira inicialização, o Docker fará o download da imagem necessária e preparará o ambiente. Esse processo pode demorar, dependendo da velocidade da internet e do computador.

Para acompanhar os registros de inicialização (logs), execute:

```bash
docker compose logs -f
```

Para sair da visualização dos logs, pressione **Ctrl + C**. Isso não encerra o container.

## 8. Acessar o ComfyUI

Quando o container estiver em execução:

1. Abra o Docker Desktop.
2. Entre em **Containers**.
3. Localize o container do ComfyUI.
4. Confira se ele está em execução.
5. Abra o navegador e acesse:

**http://localhost:8188**

Pronto! O ComfyUI estará disponível localmente.

## 9. Pastas e persistência dos arquivos

O projeto utiliza pastas locais para armazenar os arquivos e os dados de trabalho:

| Pasta | Finalidade |
|---|---|
| `run` | Arquivos de execução e instalação do ComfyUI, conforme a configuração do projeto. |
| `basedir` | Diretório de dados utilizado pelo projeto, incluindo modelos, imagens, workflows e outros arquivos conforme a configuração. |

Os modelos de IA podem ocupar bastante espaço em disco. Reserve espaço adicional para checkpoints, LoRAs e demais arquivos que forem adicionados.

## 10. Comandos úteis

Execute estes comandos no Ubuntu, dentro da pasta `ComfyUI-Nvidia-Docker`.

**Iniciar o serviço:**

```bash
docker compose up -d
```

**Parar o serviço:**

```bash
docker compose down
```

**Reiniciar o serviço:**

```bash
docker compose restart
```

**Ver os logs em tempo real:**

```bash
docker compose logs -f
```

**Ver o estado dos containers:**

```bash
docker compose ps
```

---

## Referências

- [Repositório ComfyUI-Nvidia-Docker](https://github.com/mmartial/ComfyUI-Nvidia-Docker)
- [Guia do projeto para Windows com WSL 2](https://github.com/mmartial/ComfyUI-Nvidia-Docker/wiki/Windows:-WSL2)
- [Documentação do Docker Desktop com WSL 2](https://docs.docker.com/desktop/features/wsl/)
- [Documentação NVIDIA sobre CUDA no WSL](https://docs.nvidia.com/cuda/wsl-user-guide/)

## Observações

- Este guia considera Windows 11, Docker Desktop 4.98 e NVIDIA GeForce RTX 4050.
- O teste de GPU deve ser concluído com sucesso antes da instalação do ComfyUI.
- A primeira inicialização pode levar mais tempo por causa do download das imagens e dependências.
- Os nomes dos containers, portas e diretórios dependem da configuração vigente no repositório original. Consulte o `compose.yaml` do projeto caso o mantenedor altere esses itens.
