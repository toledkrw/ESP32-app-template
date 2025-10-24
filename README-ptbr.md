# ESP32 Project Setup with Docker and Devcontainers

## Índice
1. [Introdução](#1-introdução)
2. [Pré-requisitos](#2-pré-requisitos)
    - [Instalação de drivers ESP32](#instalação-de-drivers-esp32)
3. [Configurando o Ambiente](#3-configurando-o-ambiente)
    - [WSL (Windows Subsystem for Linux)](#wsl)
    - [Docker](#docker)
    - [VSCode](#vscode)
        - [Extensão ESP-IDF](#-esp-idf-extension)
        - [Extensão Dev Containers](#-devcontainers-extension)
4. [Encaminhamento de USB](#4-encaminhamento-de-usb)
    - [Windows para WSL](#windows-para-wsl)
    - [WSL para Container Docker](#wsl-para-container-docker)
    - [Desfazendo o Encaminhamento de USB](#desfazendo-o-encaminhamento-de-usb)
5. [Clonando o Projeto](#5-clonando-o-projeto)
6. [Estrutura do Projeto](#6-estrutura-do-projeto)
7. [Build e Deploy](#7-build-e-deploy)
8. [Dicas e Solução de Problemas](#8-dicas-e-solução-de-problemas)
9. [Referências](#9-referências)

---

## 1. Introdução
Este guia tem como objetivo detalhar o processo de configuração de um ambiente de desenvolvimento moderno e reprodutível para projetos com ESP32, utilizando ferramentas como Docker, Devcontainers e WSL. Ao invés de focar em um projeto específico, o foco está em padronizar e simplificar a preparação do ambiente, tornando mais fácil para qualquer desenvolvedor iniciar, colaborar e manter projetos ESP32, independentemente do sistema operacional utilizado. Aqui você encontrará instruções passo a passo para instalar drivers, configurar o WSL, Docker, VSCode e extensões essenciais, além de dicas para forwarding de USB e solução de problemas comuns.

## 2. Pré-requisitos
- Conhecimento básico de elétrica e eletrônica
- Conhecimento de programação
    - C & C++
    - Shell
    - Python
- ESP32
- A KOMPIUTAR 🖥️
- Cabo USB
- Conexão com a Internet

### Instalação de drivers ESP32
Para que o ESP32 seja reconhecido corretamente pelo seu computador, pode ser necessário instalar o driver USB para UART, especialmente em placas baseadas no chip CP210x (Silicon Labs).

### Windows
1. **Identifique o dispositivo:**  
    Ao conectar o ESP32, se ele não for reconhecido, aparecerá no Gerenciador de Dispositivos como `CPxxxx USB to UART Bridge Controller` (por exemplo, `CP2102 USB to UART Bridge Controller`).
2. **Baixe o driver:**  
    Acesse o site oficial da Silicon Labs:  
    [https://www.silabs.com/developer-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads](https://www.silabs.com/developer-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads)
3. **Instale o driver:**  
    - Baixe o instalador correspondente ao seu sistema operacional (Windows 10/11, 64 ou 32 bits).
    - Execute o instalador e siga as instruções na tela.
4. **Atualize o driver manualmente (se necessário):**  
    - No Gerenciador de Dispositivos, clique com o botão direito no dispositivo `CPxxxx USB to UART Bridge Controller` e selecione **Atualizar driver**.
    - Escolha **Procurar software de driver no computador** e aponte para a pasta onde o driver foi extraído/instalado.
5. **Verifique a instalação:**  
    Após a instalação, o dispositivo deve aparecer como uma porta COM (por exemplo, `COM3`) e estar pronto para uso.

### Linux
Na maioria das distribuições Linux modernas, o driver para CP210x já está incluído no kernel. Para verificar:

1. **Conecte o ESP32 via USB.**
2. **Verifique se o dispositivo foi reconhecido:**  
    Execute no terminal:
    ```bash
    dmesg | grep tty
    ```
    Procure por algo como `/dev/ttyUSB0` ou `/dev/ttyACM0`.
3. **Se não for reconhecido:**  
    - Certifique-se de que o módulo está carregado:
      ```bash
      lsmod | grep cp210x
      ```
    - Caso não esteja, carregue manualmente:
      ```bash
      sudo modprobe cp210x
      ```
    - Se ainda assim não funcionar, verifique se seu usuário pertence ao grupo `dialout`:
      ```bash
      sudo usermod -aG dialout $USER
      ```
      Depois, reinicie a sessão.
4. **Referência:**  
    [Drivers Silicon Labs - Documentação](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)

Com o driver instalado, o ESP32 estará pronto para comunicação serial tanto no Windows quanto no Linux.


## 3. Configurando o Ambiente

### WSL
Para instalar o WSL (Windows Subsystem for Linux) manualmente, siga os passos abaixo:

1. **Habilite o recurso WSL:**
    Abra o PowerShell como administrador e execute:
    ```powershell
    dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
    ```
2. **Habilite a Máquina Virtual:**
    Ainda no PowerShell, execute:
    ```powershell
    dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
    ```
3. **Reinicie o computador.**
4. **Instale o kernel do WSL:**
    Baixe e execute o instalador do kernel atualizado em:  
    [https://aka.ms/wsl2kernel](https://aka.ms/wsl2kernel)
5. **Defina o WSL 2 como padrão (opcional, mas recomendado):**
    No PowerShell, execute:
    ```powershell
    wsl --set-default-version 2
    ```
6. **Instale uma distribuição Linux:**
    Acesse a Microsoft Store, pesquise por "Ubuntu" (ou outra distribuição de sua preferência) e instale.
7. **Configure a distribuição:**
    Após a instalação, abra a distribuição pelo menu iniciar e siga as instruções para criar um usuário e senha.

**Referência:**  
[Guia oficial de instalação manual do WSL](https://learn.microsoft.com/pt-br/windows/wsl/install-manual)

### Docker
Para instalar o Docker, siga os passos abaixo conforme seu sistema operacional:

#### Windows
1. **Acesse o site oficial:**  
    [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. **Baixe o instalador do Docker Desktop.**
3. **Execute o instalador:**  
    Siga as instruções na tela para concluir a instalação.
4. **Reinicie o computador se solicitado.**
5. **Verifique a instalação:**  
    Abra o terminal (cmd, PowerShell ou WSL) e execute:
    ```bash
    docker --version
    ```
    O comando deve retornar a versão instalada do Docker.

#### Linux (Ubuntu)
1. **Atualize os pacotes:**
    ```bash
    sudo apt update
    ```
2. **Instale dependências:**
    ```bash
    sudo apt install apt-transport-https ca-certificates curl software-properties-common
    ```
3. **Adicione o repositório oficial do Docker:**
    ```bash
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
4. **Instale o Docker:**
    ```bash
    sudo apt update
    sudo apt install docker-ce
    ```
5. **Verifique a instalação:**
    ```bash
    docker --version
    ```
6. **(Opcional) Adicione seu usuário ao grupo docker:**
    ```bash
    sudo usermod -aG docker $USER
    ```
    Depois, reinicie a sessão para aplicar a alteração.

**Referência:**  
[Documentação oficial do Docker](https://docs.docker.com/get-docker/)

### VSCode
Para instalar o Visual Studio Code (VSCode), siga os passos abaixo:

1. **Acesse o site oficial:**  
    [https://code.visualstudio.com/](https://code.visualstudio.com/)
2. **Baixe o instalador:**  
    Escolha a versão adequada para o seu sistema operacional (Windows ou Linux).
3. **Execute o instalador:**  
    Siga as instruções na tela para concluir a instalação.
4. **(Opcional) Instale o VSCode via linha de comando:**  
    - **Windows:**  
      ```powershell
      winget install Microsoft.VisualStudioCode
      ```
    - **Ubuntu/Linux:**  
      ```bash
      sudo snap install --classic code
      ```
5. **Verifique a instalação:**  
    Abra o VSCode e confirme que está funcionando corretamente.

Recomenda-se instalar também a extensão "Remote - Containers" para trabalhar com Devcontainers.

#### - ESP-IDF Extension
Para instalar a extensão ESP-IDF no VSCode, siga os passos abaixo:

1. **Abra o VSCode.**
2. **Acesse a aba de extensões:** Clique no ícone de extensões na barra lateral esquerda ou pressione `Ctrl+Shift+X`.
3. **Pesquise por "ESP-IDF":** No campo de busca, digite `ESP-IDF`. Ou use o link direto: [https://marketplace.visualstudio.com/items?itemName=espressif.esp-idf-extension](https://marketplace.visualstudio.com/items?itemName=espressif.esp-idf-extension)
4. **Selecione a extensão "Espressif IDF":** Clique sobre a extensão desenvolvida pela Espressif Systems.
5. **Clique em "Instalar".**

Após a instalação, siga as instruções exibidas para configurar o ambiente ESP-IDF, incluindo a instalação de dependências e configuração do Python, caso necessário.

Essa extensão facilita o desenvolvimento, build, flash e debug de projetos ESP32 diretamente pelo VSCode.
Você precisará instalar em um "sys path" a versão do IDF que utilizará, porém, no desenvolvimento através de um devcontainer, isso será feito automaticamente.


#### - Devcontainers Extension
Para instalar a extensão "Dev Containers" no VSCode, siga os passos abaixo:

1. **Abra o VSCode.**
2. **Acesse a aba de extensões:** Clique no ícone de extensões na barra lateral esquerda ou pressione `Ctrl+Shift+X`.
3. **Pesquise por "Dev Containers":** No campo de busca, digite `Dev Containers`. Ou utilize o link direto: [https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
4. **Selecione a extensão "Dev Containers" (Microsoft):** Clique sobre a extensão desenvolvida pela Microsoft.
5. **Clique em "Instalar".**

Após a instalação, você poderá abrir pastas ou projetos dentro de containers de desenvolvimento, facilitando a configuração de ambientes isolados e reprodutíveis para o seu projeto ESP32.

## 4. Encaminhamento de USB

### Windows para WSL
Para que o ESP32 conectado via USB seja acessível dentro do WSL, é necessário realizar o "bind" do dispositivo usando o utilitário `usbipd`. Siga os passos abaixo:

> Antes de listar os dispositivos, instale o utilitário `usbipd` no Windows, caso ainda não tenha feito isso. Execute no PowerShell:
> ```powershell
> winget install usbipd
> ```
1. **Liste os dispositivos USB disponíveis no Windows:**
    No PowerShell (fora do WSL), execute:
    ```powershell
    usbipd list
    ```
    Procure pelo dispositivo correspondente ao ESP32 (geralmente identificado como "CP210x" ou "Silicon Labs").
2. **Anote o BUSID e VID:PID do dispositivo** 
    Exemplo: `1-3` e `A0B1:0A1B`).
3. **Anexe o dispositivo ao WSL:**
    Ainda no PowerShell, execute:
    ```powershell
    usbipd attach --busid <ID> --wsl
    ```
    Substitua `<ID>` pelo identificador anotado anteriormente.
4. **Verifique no WSL:**
    No terminal do WSL, execute e verifique se o dispositivo aparece como um USB tendo a mensagem:
    ```bash
    dmesg | tail -n 10
    "usb 1-1: cp210x converter now attached to ttyUSB0"
    ```
5. **Atribua o driver manualmente, se necessário**
    Caso não tenha a mensagem anterior ou sequer tenha algum dispositivo ```ttyUSB```:
    ```bash
    ls /dev/ttyUSB*
    ```
    Você precisará atribuir o driver manualmente ao dispositivo, e então monta-lo em um nó USB. Lembre-se do seu "VID:PID" e substitua-os no comando à seguir:
    ```bash
    modprobe cp210x
    echo <VID> <PID> | sudo tee /sys/bus/usb-serial/drivers/cp210x/new_id
    ```
    Confira novamente se o dispositivo foi montado em algum ```ttyUSB```:
    ```bash
    dmesg | tail -n 10
    "usb 1-1: cp210x converter now attached to ttyUSB0"
    ```    

**Referência:**  
[Conectar dispositivos USB ao WSL - Documentação Microsoft](https://learn.microsoft.com/pt-br/windows/wsl/connect-usb#attach-a-usb-device)


### WSL para Container Docker
Quando você utiliza o `usbipd` para fazer o bind do dispositivo USB à sua instância WSL (por exemplo, `Ubuntu` ou `docker-desktop`), o dispositivo USB se torna visível para o kernel do WSL. Como o Docker Desktop para Windows executa containers Linux dentro do WSL 2, os containers podem acessar dispositivos USB expostos pelo WSL, desde que o dispositivo seja explicitamente mapeado para o container usando a opção `--device` no `devcontainer.json`. Ou seja, o forwarding feito pelo `usbipd` torna o dispositivo disponível para o WSL, mas cada container ainda precisa receber permissão para acessar o dispositivo (por exemplo, `/dev/ttyUSB0`) via configuração no `devcontainer.json`. Não é necessário configurar drivers adicionais dentro do container se o dispositivo já estiver funcional no WSL, mas o mapeamento do device é obrigatório para acesso pelo container.
```json
"runArgs": [
		"--device=/dev/ttyUSB0:/dev/ttyUSB0",
		"--privileged"
	]
```

### Desfazendo o Encaminhamento de USB

Para desfazer o encaminhamento (forwarding) de um dispositivo USB conectado ao WSL, utilize o comando:
```powershell
usbipd detach --busid <ID>
```
O parâmetro `<ID>` deve ser substituído pelo identificador do barramento USB do dispositivo que foi previamente encaminhado.
Esse comando desconecta o dispositivo USB do ambiente WSL, tornando-o novamente disponível para o Windows ou outros sistemas.

> Atente-se que possívelmente será necessário realizar o processo de encaminhamento USB e montagem de nó caso troque seu dispositivo de porta USB, uma vez que seu ```BUSID``` irá mudar.

## 5. Clonando o Projeto
<!-- TODO -->

## 6. Estrutura do Projeto
A estrutura deste projeto é simples e segue o padrão recomendado para projetos ESP32 com CMake:

```
├── CMakeLists.txt           # Arquivo principal de configuração do CMake para o projeto
├── main
│   ├── CMakeLists.txt       # Configuração específica do CMake para o código-fonte principal
│   └── main.c               # Código-fonte principal da aplicação
└── README.md                # Este arquivo de documentação
```

- **CMakeLists.txt**: Arquivo de configuração do CMake na raiz, responsável por definir as regras de build do projeto.
- **main/**: Diretório que contém o código-fonte principal da aplicação, incluindo seu próprio CMakeLists.txt e o arquivo `main.c`.
- **README.md**: Documentação do projeto, com instruções de uso, configuração e informações relevantes.

Essa organização facilita a manutenção, a escalabilidade e o entendimento do projeto, seguindo as boas práticas para projetos ESP32 baseados em CMake.

## 7. Build e Deploy
<!-- TODO -->

## 8. Dicas e Solução de Problemas
<!-- TODO -->

## 9. Referências

- [Documentação oficial do ESP-IDF](https://docs.espressif.com/projects/esp-idf/pt/latest/)
- [Guia de instalação manual do WSL (Microsoft)](https://learn.microsoft.com/pt-br/windows/wsl/install-manual)
- [Conectar dispositivos USB ao WSL (Microsoft)](https://learn.microsoft.com/pt-br/windows/wsl/connect-usb)
- [Drivers USB to UART Bridge (Silicon Labs)](https://www.silabs.com/developer-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads)
- [Documentação oficial do Docker](https://docs.docker.com/get-docker/)
- [Visual Studio Code - Site oficial](https://code.visualstudio.com/)
- [Extensão ESP-IDF para VSCode](https://marketplace.visualstudio.com/items?itemName=espressif.esp-idf-extension)
- [Extensão Dev Containers para VSCode](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
- [Repositório Awesome ESP32 (recursos e projetos)](https://github.com/agucova/awesome-esp32)
- [Comunidade ESP32 no GitHub](https://github.com/espressif/esp-idf)
- [Docker Desktop para Windows](https://www.docker.com/products/docker-desktop/)