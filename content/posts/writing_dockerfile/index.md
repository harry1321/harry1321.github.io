---
sys:
  pageId: "2af1f336-f350-8079-9d32-cce32aadc1c7"
  createdTime: "2025-11-18T06:58:00.000Z"
  lastEditedTime: "2025-11-21T09:19:00.000Z"
  propFilepath: "posts/writing_dockerfile/index.md"
title: "Writing Dockerfile"
date: "2025-11-20T00:00:00Z"
description: ""
tags:
  - "🏷️Docker"
  - "🏷️DevOps"
categories:
  - "📱 技術"
author: "Harry Yang"
draft: false
url: "/posts/writing-dockerfile"
section: "posts"
---

# Writing Dockerfile


在[上一篇文章](/posts/docker-basic/index.md)，提到映像檔是容器啟動的基底，雖然可以從公開的 registry 拉取建置完成的映像檔。但是有時候可能沒有符合開發需求的檔案，或是想要客製化某些功能，這時候就需要自行撰寫 Dockerfile 來建立映像檔。


Dockerfile 是一個純文字的檔案，包含一連串的指令來告訴 Docker 建造映像檔。可以想像 Dockerfile 是一張映像檔的建築藍圖，Docker 則是幫忙蓋房子的工人。在建造映像檔時，因為Docker image 在本質上是由多個唯讀檔案組成的分層結構，所以其實在 Dockerfile 的每一行指令將會變成一層唯讀層。愈多指令會產生更多的層數，當層數愈多，將會增加映像檔大小、降低存取效率並影響部署速度。所以將 Dockerfile 寫的簡潔是個重要的學問，在後續也會提到壓縮層數的技巧。


首先是關於檔案的名稱，一般來說能夠直接使用 Dockerfile 這個檔名，因為在建立 image 的指令中不用指明特定的檔案，Docker 就會預設尋找在目前工作目錄下名為 Dockerfile 的檔案。但是有時候一個專案內可能需要多個應用程式，這時候為了區別用途有兩種解決方式。一是使用不同名稱來區別，因為 Docker 並沒有硬性規定檔案名稱，不過一般的命名規則推薦使用<application_name>.Dockerfile。這時候只要在 docker buildx build 指令加上 --file 標籤，就能指示建立對應的應用程式容器行。第二種方法是將檔案存放在不同的目錄下，同樣也需要使用 --file 標籤。當然也能夠結合這兩種做法，重點是當檔案名稱不是 Dockerfile 時，就需要明確指示要使用的檔案名稱和路徑。


## Basic


Dockerfile 檔案中大致上有幾個重要的執行指令：

- [FROM](#from)
- [WORKDIR](#workdir)
- [COPY](#copy)
- [RUN](#run)
- [EXPOSE](#expose)
- [CMD](#cmd)

使用這些指令我們能輕鬆打造一個簡單的容器，下面將會使用一個範例說明，過程中這些指令實際執行內容，來建構一個應用程式。Docker 中還有更多指令，來幫助使用者完成更進階的開發需求，詳細的資訊可以前往下面的連結有更多詳細介紹。編寫這些指令是沒有區分大小寫的要求，不過實務上來講為了區分指令和參數，通常指令關鍵詞會全部使用大寫。


{{< alert "lightbulb" >}}
[https://docs.docker.com/reference/dockerfile/](https://docs.docker.com/reference/dockerfile/)
{{< /alert >}}


首先透過一個簡單的範例來說明，假設現在想要用一個 python 來執行啟用一個 flask 網頁。先試想在本地端開發需要用到什麼工具，可能是 runtime、依賴套件、程式碼等等，這些同樣需要出現在容器中，只是現在要編寫的是要如何告訴 Docker 如何安裝和執行。


### Quick Example


這是目前的工作目錄結構，其中包含 app 程式碼、python 依賴列表以及Dockerfile。


```markdown
project
├── flask
│   └── app.py       # Flask 應用程式
├── requirements.txt # Python 依賴列表
└── Dockerfile
```


```python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World'

if __name__ == "__main__":
    app.run(debug=True, host='0.0.0.0', port=5000)
```


```docker
FROM python:3.14-slim

WORKDIR /app

# install dependices
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# install app
COPY app .

# final configuration
EXPOSE 5000
CMD ["python", "app.py"]
```


### Instructions


接下來將會依序介紹這幾個重要的指令，來幫助理解以上的 Dockerfile 實際做了什麼，以及要如何使用來完成不同的開發需求。

- #### FROM

	在 Dockerfile 中可以看到使用 `FROM` 作為開頭，在每一個 Dockerfile 一定都是如此，而且一定至少要有一個。FROM 是指定一個基底映像檔，並在那之上開始建立新的映像檔，範例中使用的是python 3.14 的 silm 版本。一般開發目的，會使用公開倉庫上現有的環境 image 來做基底直接修改，通常不會希望每次都要重新設計執行環境的 image。

	除了指定基底映像，通常需要指定一個版本，來穩定開發和部署流程。在公開的 regisry 中，同樣名稱的映象有很多標籤和版本，一般的應用如果沒有特定需求，使用愈小的記憶體空間會是較佳的選擇，因為要盡可能減少部署的負擔。


	```docker
	# give a name to build stage can be used in subsequent stages
	FROM <image> [AS <name>]
	# specify a tag of base image if omitted the builder uses a latest tag by default
	FROM <image> [:<tag>] [AS <name>]
	```

- #### WORKDIR

	指定在建造 image 過程中的工作目錄，執行`COPY` `ADD` `RUN` `CMD` `ENTRYPOINT` 時，會依據這個的工作目錄來執行。如果指定的目錄不存在，系統將會自動創建，但沒有特別指定預設會是在`/` 。所以為了避免在建造過程產生錯誤和混亂，最好在執行其他指令前先明確指定工作目錄。

	```docker
	# this is an absolute path
	WORKDIR /path
	# this is a relative path
	WORKDIR myapp
	```

- #### COPY

	複製來源路徑的檔案、目錄，將其寫入映像檔檔案系統中的指定位置。目的地路徑則是可以是絕對路徑或 WORKDIR 的相對路徑。如目標路徑不存在，Docker 會自動建立所需的目錄。

	```docker
	# copy source to destination
	COPY [OPTIONS] <src1> <src2> ... <dest>
	COPY [OPTIONS] ["<src>", ... "<dest>"]
	```


	**Source**


	來源路徑是依照 build context 的相對路徑，如 app/、app/app.py、requirements.txt，必須為位於 build context 之中。如果路徑包含 `../..`將會被 Docker 忽略；或是在結尾有斜線號(trailing slashes) 如`source/`，也會被忽略視為`source`。


	如果來源是一個路徑，那 COPY 只會複製該目錄下的內容，目錄本身不會被複製。其中來源支援使用萬用字元，如 `*.txt` `index.?s`來匹配路徑中的檔案。


	**Destination**


	目標路徑若以斜線`/`開頭，例如 `/app/`，則視為容器內的絕對路徑；不以斜線 `/`開頭，例如`app/`，會被視為相對於當前 WORKDIR 的相對路徑。


	而使用結尾斜線(trailing slashes)如`/app/`，Docker 視做目標目錄，將來源檔案或目錄內容複製到這個目錄內。如果沒有使用 trailing slashes 結尾，且來源是單一檔案，Docker 會視目標路徑做為新的檔名，來儲存要複製的檔案。當複製單一檔案時，我們通常希望保留檔名，所以會寫成 /app/。當複製目錄時，我們通常不希望丟失目錄名，所以會寫成 /app。


	```docker
	# copy single file and we don't want to rename the file name
	# result app/file.txt
	COPY file.txt app/
	# the destination will be viewed as parent directory for source directory
	# result app/directory/files
	COPY directory app
	```

- #### RUN

	被用作在建立 image 時執行 shell command，來幫助建立資料夾或安裝依賴項目等等。每一個 `RUN` 指令會在現有映像檔之上加入新的一層，指令於該層被執行並提交結果。有兩種格式，分別是 shell 形式和 exec 形式，在實務上通常使用 shell form，因為可以透過 newline escape 增加過長的指令的可讀性。


	> exec form 是使用 JSON 陣列語法，陣列中的每個元素是一個命令、標誌或參數，並且必須使用雙引號將元素括起來，禁止使用單引號。


	```docker
	# shell form
	RUN <command>
	RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'

	#  exec form
	RUN ["executable", "param1", "param2"]
	RUN ["/bin/bash", "-c", "echo hello"]
	```

- #### EXPOSE

	EXPOSE 主要是宣告容器內應用程式使用的監聽埠，並作為一種文件記錄，告知映像檔的使用者，讓使用者知道要與這個服務互動，應該注意哪個埠號。實際開通埠號映射，需要在執行 `docker run`時加入參數，如: `-p 5000:5000`，才能讓主機外部進入容器。

- #### CMD

	CMD 是用來設定啟動容器的預設行為，將在啟動容器的時候執行。一個 `Dockerfile` 中預設只會有一個 `CMD` 指令，如果出現多次，則僅有最後一個會產生作用。與`RUN`同樣也有 shell 形式和 exec 形式，但不同之處在於`RUN`是在建立過程中執行的指令，`CMD` 則是容器要啟動時執行的指令。


	CMD 設定的預設行為可以是一個可執行檔(executable)，也可以省略執行檔只設定參數。但在後者的情況下，必須同時指定 ENTRYPOINT 指令。


	```docker
	# Exec form
	CMD ["executable", "param1", "param2"]

	# Exec form set as default parameters
	CMD ["param1", "param2"]

	# Shell form
	CMD command param1 param2
	```


## Build Context


在創建 Image 輸入的命令中 `docker buildx build -t tags:version -f filepath .`，包含三個引數`-t tags:version`、`-f filepath` 和`.`。其中 `.` 是 Docker builder 創建過程中存取檔案的路徑依據，也限定了它只能訪問這個指定路徑及其下的所有子路徑。


可以用作 build context 的路徑包含：

- 本地的相對或絕對路徑
- 遠端的 Git 專案或壓縮檔

當一個專案中開發多個應用程式，並封裝在不同的容器中，需要透過 `--file` 標籤來指定對應的 Dockerfile。並且需要注意提供的 build context，能否讓 builder 依照 Dockerfile 的指令順利建立 image。


```markdown
project
├── app1             # 應用程式1
│   ├─  Dockerfile
│   └── app1.py
├── app2             # 應用程式2
│   ├─  Dockerfile
│   └── app2.py
├── requirements.txt # 依賴列表
└── config           # 設定檔
```


假設現在要啟動 app2，並載入在 config 中的設定檔。首先需要找到對應的 Dockerfile: ./app2/Dockerfile，接著注意要將 config 包進 build context 中，而目前的工作目錄是在專案根目錄。整合全部需求，建立 app2 的 Docker 指令會是：


```shell
docker buildx build -f app2/Dockerfile .
```


如果切換 Docker 目前的工作目錄，或是將 build context 改為 `project/app2`，都有可能造成建構失敗。


執行 `docker build` 時，Docker Client 會將整個 build context 壓縮傳輸給 Docker Daemon。多數人習慣將 Dockerfile 放在專案的最上層，而在專案目錄中可能會包含創建 Image 時不需要的檔案，可能是幾百 MB 甚至幾 GB 的大小，會導致創建的速度降低。改善的方法其中一種是將 build context 放在適合的路徑，另一種是使用 `.dockerignore`，設定不要載入的檔案，讓 Docker 直接忽略。


> 如果使用多個 Dockerfile，需要將 `.dockerignore`放在相同的目錄下，並命名為`APPNAME.Dockerfile.dockerignore` 。


## ENTRYPOINT vs CMD


`ENTRYPOINT`


entrypoint 是容器啟動後執行的一段命令或腳本，將此 container 作為一個可執行檔(executable)。也就是預期容器專門用於即執行特定的任務或服務，與執行可能需要手動輸入命令的通用環境不同，如 shell。因為 entrypoint 具有較高的優先層級，不容易被輕易覆蓋，所以在確定容器的用途後，會使用 entrypoint 而非 CMD。若使用 entrypoint 作為容器啟動的腳本，CMD 的用途會為成傳遞引數給啟動腳本。


> 若在 Docker CLI 的指令中輸入引數，將會覆蓋掉 Dockerfile 中`CMD`定義的引數。但在 Dockerfile 中定義的 `ENTRYPOINT`則不受影響，除非透過 `--entrypoint` 重新定義，才會被覆蓋掉。


`ENTRYPOINT` 和`CMD` 同樣都能使用 shell form 和 exec form，這使得兩者的交互關係變得複雜。如果使用 shell form 形式的 `ENTRYPOINT` 指令，將會作為 `/bin/sh -c` 的 subcommand 來執行，並且會忽略任何 `CMD` 以及 command line 的引數設定。[官網](https://docs.docker.com/reference/dockerfile/#understand-how-cmd-and-entrypoint-interact)也給出更詳細的交互關係，實務上建議都使用 exec form 來撰寫指令，這樣是最能確保容器依照預期的設定執行。


這兩個指令的差異，讓各自在使用上有不同的適用時機。

- ENTRYPOINT 適用時機是需要固定容器啟動後的行為，要執行主要應用程序時使用。有不容易被覆蓋的特性，例如啟動一個 web 伺服器、一個 database。
- CMD 則有被使用者輕鬆地覆寫的特性，在執行 command line 時提供了靈活性。適用在提供預設引數、簡化 ENTRYPOINT，專注於主應用程式的執行，而將可變動的行為放在 CMD 中。

{{< alert "lightbulb" >}}
如果還有不理解的地方，我認為[這篇文章](https://mihirpopat.medium.com/understanding-dockerfile-entrypoint-vs-cmd-whats-the-real-difference-ab94df1e4bdf)提供更詳盡的說明。
{{< /alert >}}

---

# Upload Image to Registry


當完成一個 Dockerfile 後，通常會將映像檔作為應用程式標準化的交付物。而現在 CI/CD 的發展中，這種標準化結合雲端服務已成為當前的標準作業模式。這時候需要一個集中、可靠的儲存空間存放映像檔。用來存放映像檔的 image registry 是 CI/CD 流程中的核心環節，提供在不同環境(開發、測試、生產)下，能快速部署應用程式的能力。


當專案推送至 GitHub 專案後，可以應用 GitHub Actions 來觸發整個 CI/CD流程。整個流程通常是，本地完成開發、測試，推送專案之後觸發 CI/CD。接下來在 GitHub Actions 的 runner 上進行測試，完成遠端測試後再推送到 Docker Hub，這樣就算完成 CI 的部分。[官方](https://docs.github.com/en/actions/tutorials/publish-packages/publish-docker-images#publishing-images-to-docker-hub)有提供如何應用 GitHub Actions 結合 Docker Hub 的 CI 流程，在這裡就不多作介紹。


```yaml
# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

# GitHub recommends pinning actions to a commit SHA.
# To get a newer version, you will need to update the SHA.
# You can also reference a tag or branch, but the action may change without warning.

name: Publish Docker image

on:
  release:
    types: [published]

jobs:
  push_to_registry:
    name: Push Docker image to Docker Hub
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
      attestations: write
      id-token: write
    steps:
      - name: Check out the repo
        uses: actions/checkout@v5

      - name: Log in to Docker Hub
        uses: docker/login-action@f4ef78c080cd8ba55a85445d5b36e214a81df20a
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@9ec57ed1fcdbf14dcef7dfbe97b2010124a938b7
        with:
          images: my-docker-hub-namespace/my-docker-hub-repository

      - name: Build and push Docker image
        id: push
        uses: docker/build-push-action@3b5e8027fcad23fda98b2e3ac259d8d67585f671
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}

      - name: Generate artifact attestation
        uses: actions/attest-build-provenance@v3
        with:
          subject-name: index.docker.io/my-docker-hub-namespace/my-docker-hub-repository
          subject-digest: ${{ steps.push.outputs.digest }}
          push-to-registry: true

```


---


參考來源


[https://docs.docker.com/build/concepts/context/](https://docs.docker.com/build/concepts/context/)


[https://docs.docker.com/reference/dockerfile/](https://docs.docker.com/reference/dockerfile/)


[https://ragin.medium.com/docker-what-it-is-how-images-are-structured-docker-vs-vm-and-some-tips-part-1-d9686303590f](https://ragin.medium.com/docker-what-it-is-how-images-are-structured-docker-vs-vm-and-some-tips-part-1-d9686303590f)


[https://hackmd.io/@tienyulin/docker3](https://hackmd.io/@tienyulin/docker3)


[https://blog.ewocker.com/blog/container-docker/02](https://blog.ewocker.com/blog/container-docker/02)


[https://cutejaneii.gitbook.io/docker/docker-registry/shang-chuan-dao-gong-you-cang-ku](https://cutejaneii.gitbook.io/docker/docker-registry/shang-chuan-dao-gong-you-cang-ku)
