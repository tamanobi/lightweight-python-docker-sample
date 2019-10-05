# lightweight-python-docker-sample

https://lekum.org/post/multistage-dockerfile/ の記事に従ってPythonのDocker Imageのファイルサイズを削減してみる。記事のアプローチはマルチステージビルド。このとき、私はGo言語でしかマルチステージビルドを使ったことがなかった。

記事を読むと、/root/.cacheをコピーすることで軽量しているようだった。Pythonは/root/.cacheにpip installしたときにさまざまなデータを放り込むらしい。

pycryptoライブラリは、python:3-alpineやpython:3-slimではインストールできない。比較のために、マルチステージビルドした場合としなかった場合の結果を次に示す。

```
$ docker image ls | grep '^test\|REPO'
REPOSITORY                                                TAG                 IMAGE ID            CREATED             SIZE
test-large                                                latest              13bfbb4705cc        5 minutes ago       926MB
test                                                      latest              d85137a5222a        12 minutes ago      107MB
```

なんと、1/9まで削減できている。

```
$ docker history test
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
d85137a5222a        13 minutes ago      /bin/sh -c pip install -r requirements.txt &…   7.02MB
bb7154af4c99        13 minutes ago      /bin/sh -c #(nop) COPY file:23432deec6e2c2a4…   9B
a99765c3e29f        13 minutes ago      /bin/sh -c #(nop) COPY dir:0f89174a79a5351b1…   983kB
39fb80313465        5 weeks ago         /bin/sh -c #(nop)  CMD ["python3"]              0B
<missing>           5 weeks ago         /bin/sh -c set -ex;   wget -O get-pip.py "$P…   6.24MB
<missing>           5 weeks ago         /bin/sh -c #(nop)  ENV PYTHON_GET_PIP_SHA256…   0B
<missing>           5 weeks ago         /bin/sh -c #(nop)  ENV PYTHON_GET_PIP_URL=ht…   0B
<missing>           5 weeks ago         /bin/sh -c #(nop)  ENV PYTHON_PIP_VERSION=19…   0B
<missing>           6 weeks ago         /bin/sh -c cd /usr/local/bin  && ln -s idle3…   32B
<missing>           6 weeks ago         /bin/sh -c set -ex  && apk add --no-cache --…   86.4MB
<missing>           6 weeks ago         /bin/sh -c #(nop)  ENV PYTHON_VERSION=3.7.4     0B
<missing>           6 weeks ago         /bin/sh -c #(nop)  ENV GPG_KEY=0D96DF4D4110E…   0B
<missing>           6 weeks ago         /bin/sh -c apk add --no-cache ca-certificates   551kB
<missing>           6 weeks ago         /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B
<missing>           6 weeks ago         /bin/sh -c #(nop)  ENV PATH=/usr/local/bin:/…   0B
<missing>           6 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>           6 weeks ago         /bin/sh -c #(nop) ADD file:fe64057fbb83dccb9…   5.58MB
```

```
$ docker history test-large
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
13bfbb4705cc        7 minutes ago       /bin/sh -c pip install -r requirements.txt &…   8.22MB
eeac6b736730        14 minutes ago      /bin/sh -c #(nop) COPY file:a8e86882d87449a7…   9B
02d2bb146b3b        3 weeks ago         /bin/sh -c #(nop)  CMD ["python3"]              0B
<missing>           3 weeks ago         /bin/sh -c set -ex;   wget -O get-pip.py "$P…   6.24MB
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV PYTHON_GET_PIP_SHA256…   0B
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV PYTHON_GET_PIP_URL=ht…   0B
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV PYTHON_PIP_VERSION=19…   0B
<missing>           3 weeks ago         /bin/sh -c cd /usr/local/bin  && ln -s idle3…   32B
<missing>           3 weeks ago         /bin/sh -c set -ex   && wget -O python.tar.x…   91.1MB
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV PYTHON_VERSION=3.7.4     0B
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV GPG_KEY=0D96DF4D4110E…   0B
<missing>           3 weeks ago         /bin/sh -c apt-get update && apt-get install…   17.1MB
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV PATH=/usr/local/bin:/…   0B
<missing>           3 weeks ago         /bin/sh -c set -ex;  apt-get update;  apt-ge…   510MB
<missing>           3 weeks ago         /bin/sh -c apt-get update && apt-get install…   145MB
<missing>           3 weeks ago         /bin/sh -c set -ex;  if ! command -v gpg > /…   17.5MB
<missing>           3 weeks ago         /bin/sh -c apt-get update && apt-get install…   16.5MB
<missing>           3 weeks ago         /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>           3 weeks ago         /bin/sh -c #(nop) ADD file:770e381defc5e4a0b…   114MB
```

test-largeで使用したベースイメージがDebianである(see Dockerfile.large)ため、そのレイヤーだけが510MB消費している。そのハンデを抜いたとしても、マルチステージビルドでは107MB、マルチステージを使わない場合で426MBとなり、大幅な削減ができている。
