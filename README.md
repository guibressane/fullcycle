# Fullcycle Desafio Golang
Desafio de implementação da linguagem go em docker proposto por Fullcycle 

## Docker Fullcycle

Esse desafio é muito empolgante principalmente se você nunca trabalhou com a linguagem Go!
Você terá que publicar uma imagem no docker hub. Quando executarmos:
docker run <seu-user>/fullcycle
Temos que ter o seguinte resultado: Full Cycle Rocks!!
Se você perceber, essa imagem apenas realiza um print da mensagem como resultado final, logo, 
vale a pena dar uma conferida no próprio site da Go Lang para aprender como fazer um "olá mundo".
Lembrando que a Go Lang possui imagens oficiais prontas, vale a pena consultar o Docker Hub.
A imagem de nosso projeto Go precisa ter menos de 2MB =)

Primeiro vamos pesquisar sobre a documentação da linguagem Go:
https://go.dev/doc/tutorial/getting-started

Vamos criar uma pasta "golang" e dentro dela um arquivo "desafio-golang.go".
Nele colocaremos:
```
package main

import "fmt"

func main() {
    fmt.Println("Full Cycle Rocks!!")
}
```

Pesquisando no Docker Hub, encontramos uma imagem golang alpine.
Dentro da pasta "golang" criaremos um "Dockerfile". Vamos 
utilizar o multi-stage build para compilar a aplicação e otimizar a imagem:

```
# Iniciando uma imagem base golang:alpine
FROM golang:alpine AS builder

# definindo diretório de trabalho
WORKDIR /src

# copiando a aplicação
COPY . .

# compilando o binário e removendo informações de debug
RUN go build -ldflags '-s -w' desafio-golang.go
```

Colocamos alguns parâmetros para o linker via -ldflags
que vão ajudar a diminuir o tamanho do executável final ( -ldflags '-s -w' )
O parâmetro -s remove informações de debug do executável 
e o -w impede a geração do DWARF (Debugging With Attributed Record Formats).

Para construir a imagem usaremos "build":
```
sudo docker build -t [usuario-dockerhub]/[nome-imagem] .
```

Por exemplo:
```
docker build -t guibressane/fullcycle-inicial
```

Nessa primeira etapa já poderemos ver a imagem criada em nosso banco de imagens:
```
docker images
```

Aparecerá algo como:
```
REPOSITORY                      TAG       IMAGE ID       CREATED         SIZE
guibressane/fullcycle-inicial   latest    e159deda8049   5 seconds ago  276MB
```

Ela está com 276MB, precisamos reduzir esse tamanho, para isso
vamos compilar a imagem usando Scratch. Copiaremos nosso "Dockerfile"
num arquivo chamado "Dockerfile.prod" e acrescentaremos o seguinte
abaixo do que já existia:

```
# Iniciando com scratch
FROM scratch

# diretório de trabalho
WORKDIR /

# copiando o binário
COPY --from=builder /src / 

# executando 
CMD ["./desafio-golang"]
```

Para construir uma nova imagem usaremos o "build" mais uma vez:
```
docker build -t [usuario-dockerhub]/[nome-imagem] .
```

Em nosso exemplo vamos definir que a construção do container
seja realizada com o "Dockerfile.prod":
```
docker build -t guibressane/fullcycle -f Dockerfile.prod . 
```

Vamos olhar em nosso banco de imagens:
```
docker images
```

Aparecerá algo como:
```
REPOSITORY                      TAG       IMAGE ID       CREATED           SIZE
guibressane/fullcycle           latest    69e9f2715145   10 seconds ago   1.39MB
guibressane/fullcycle-inicial   latest    e159deda8049   7 minutes ago     276MB
```

Podemos ver que a imagem reduziu consideravelmente de 276MB para 1.39MB
Agora vamos subir nossa imagem no Docker Hub:

## Push
Para subir no Docker Hub precisamos executar o "push":
```
docker push guibressane/fullcycle
```
## Pull
Caso quisermos baixar de outra máquina ou baixar novamente em nossa máquina poderemos usar o "pull":
```
docker pull guibressane/fullcycle
```
## Run
Para executar nossa aplicação usaremos o "run":
```
docker run guibressane/fullcycle
```

## Referências
- https://go.dev/doc/tutorial/getting-started
- https://hub.docker.com/_/golang/
- https://hub.docker.com/_/scratch/

