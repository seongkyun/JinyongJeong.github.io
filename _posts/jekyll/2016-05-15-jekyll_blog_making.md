---
layout: post
title:  "Jekyll을 이용하여 Github Blog 만들기"
date:   2016-05-15 13:33:49 +0900
categories: jekyll update
comments: true
image: "/fig/default/default_post.jpg"
summary: "Jekyll을 이용한 간편하게 Github에 blog를 만드는 방법 소개"
categories: [jekyll]
---

Jekyll을 이용해서 Github에 Blog를 만들어보자. 우선 Jekyll을 설치하기 위해서는 rvm(ruby version manager)를 설치해야 한다. rvm은 ruby의 버전관리를 위해 사용된다. ruby와 jekyll 설치 시 `apt-get install`을 사용하면 안된다. 전체적인 순서는 rvm 설치 -> ruby 설치 -> jekyll 설치 -> github.io 생성 이다. 

# 1. curl/rvm(ruby version manager) 설치

````
sudo apt-get install curl
curl -sSL https://get.rvm.io | bash -s stable
````

위 코드의 설치 과정에서 만약 다음과같은 error가 뜬다면
{% highlight ruby %}
curl: (77) error setting certificate verify locations:
CAfile: /etc/pki/tls/certs/ca-bundle.crt
CApath: none
{% endhighlight %}

certificate 경로가 맞지 않는 것이다. 다음의 코드를 입력하여 경로 생성 후 `.crt` 파일을 복사해준다. 만약 `.crt`파일이 없다면 `sudo apt-get install ca-certificates`를 통해 설치한다.

````
sudo mkdir -p /etc/pki/tls/certs
sudo cp /etc/ssl/certs/ca-certificates.crt /etc/pki/tls/certs/ca-bundle.crt
````

다른 에러가 또 뜬다면 다음을 입력(아마 error 내용에 다음의 내용이 있을 것이다.)

````
command curl -sSL https://rvm.io/mpapis.asc | gpg --import -
````

rvm 스크립트를 현재 쉘에 로드 및 현재 쉘에 로드

````
source ~/.rvm/scripts/rvm
rvm requirements
````

만약 다음과 같은 error가 발생한다면(아마 한국 서버에서 받아오는데 문제가 있는것 같다.)

{% highlight ruby %}
E: Failed to fetch http://kr.archive.ubuntu.com/ubuntu/pool/main/g/gawk/gawk_4.0.1+dfsg-2.1ubuntu2_amd64.deb Could not connect to kr.archive.ubuntu.com:80 (103.22.220.133). - connect (111: Connection refused)
{% endhighlight %}

Setting의 software updates에서 Download from 을 main server 로 바꿔준 후 다시 `rvm requirements` 실행한다.

# 2. Ruby 설치

Ruby 2.1.2 설치 후 디폴트로 설정

````
rvm install 2.1.2
rvm use 2.1.2 --default
````

# 3. nodejs 설치

````
sudo apt-get install nodejs
````

# 4. Jekyll 설치 및 버전 확인

````
gem install jekyll
jekyll -v
````

여기까지 Jekyll 설치 완료.

# 5. Jekyll의 설치를 확인

Jekyll을 이용하여 기본 테마를 생성 후 서버를 동작시킨다.

````
jekyll new myblog
cd myblog
jekyll serve
````

`http://localhost:4000` 으로 들어가면 local 블로그를 확인할 수 있다.

# 6. github의 repository를 생성

[Github](http://www.github.com)으로 접속 후 새로운 repository 생성. <yourname>.github.io 로 repository 생성 후 로컬에서 home folder로 이동하여 repository 복사.

````
git clone https://github.com/<yourname>/<yourname>.github.io.git
````

폴더를  복사 후  그 폴더로 이동한 후 git 을 초기화시킨다. 초기화 시킨 후 git의 폴더에 jekyll의 기본 테마 설치

````
git init
jekyll new . --force 
````

Jekyll로 생성된 모든 파일을 git에 add해 준 후 commit, push 를 해서 github에 올려준다.

````
git add *
git commit -m "first commit"
git push -u origin master
````

첫 commit 시에 config 파일에 user.email과 user.name을 등록하라고 나오면 

````
git config user.email "이메일주소"
git config user.name "이름"
````

을 입력하여 이메일과 이름을 등록해 준다.이로써 github에 블로그가 생성되었다!

git을 clone 받은 후 처음에는 `git push -u origin master`를 모두 입력해야 하지만 한번 한 이후에는 `git push`만 입력해도 된다.

# 7. Blog 작성

블로그 작성 시 입력한 내용이 예상한대로 동작을 하는지 확인 하기 위해서 매번 github에 push하는 것은 매우 힘든 일이다. 작성을 하는 동시에 바로 확인을 하기 위해서 위에서 언급한 것과 같이 `jekyll serve`를 이용하여 서버를 동작 시킨 후 `localhost:4000`으로 접속하면 바로 수정된 내용을 확인할 수 있다. 이렇게 수정이 완료된 후 github로 push하면 편하게 포스팅을 수정할 수 있다.

