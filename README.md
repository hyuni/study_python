# 파이썬 팁 정리

## Ubuntu setting

### 서버 timezone 변경

```bash
sudo ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```

### oh-my-zsh

```bash
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
sudo apt install fonts-powerline
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

```.zsh_rc
ZSH_THEME="agnoster"

plugins=(
        git
        git-flow
        gitignore
        sudo
        colored-man-pages
        zsh-syntax-highlighting
        zsh-autosuggestions
        fzf
)

source $ZSH/oh-my-zsh.sh
```

### python 3.12

```bash
# repository 추가
sudo apt update && sudo apt upgrade -y
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update

# python 3.12 설치
sudo apt install python3.12

# default 설정
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 1
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.12 2

# 명령어 실행후 default 를 python 3.12 로 변경
sudo update-alternatives --config python3

# pip 설치 
sudo apt install python3.12-distutils
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
sudo python3.12 get-pip.py
```

### requirements.txt

- 현재 프로젝트에서 사용하고 있는 library 를 requirement.txt 로 저장

    ```bash
    pip freeze > requirements.txt
    ```

## crawling 관련

### request 를 반복하는 경우
  
  - 수집할 목록을 먼저 만들고, 상세한 내용이 필요한 경우 별도로 처리  
  
  - 필요한 경우라면 file, db 를 사용해서 반복된 것들은 필터링 처리  
  
  - time.sleep() 사용. 이왕이면 random 하게 적용되도록  

    ```python
    import time
    time.sleep(random.uniform(3, 10))
    ```
    
### BeautifulSoup 사용시

  - xPath 를 지원하지 않음. css selector 를 기본으로 사용하므로 학습이 필요  

    > [CSS Selector Reference](https://www.w3schools.com/cssref/css_selectors.php)
    > [CSS Selector Tester](https://www.w3schools.com/cssref/trysel.php)

  - web page 가 title, 내용이 분리되어 하나의 Item 을 이루고, 여러개의 Item 들로 이루어진 경우 Zip 사용

    ```python
    # css selector class="subject" or "summary"
    titles = soup.select('.subject')
    summaries = soup.select('.summary')
    for item in zip(titles, summaries):
        # item 으로 원하는 처리
        pass
    ```
    
  - Comprehension 사용
    
  예를 들어 web page 가 여러개 가 있을수 있지만, url parameter 가 규칙적 이지 않거나 별도의 링크를 통해 처리하는 경우라면  
  한 페이지를 request 한후 page 들을 리스트업 후 조건 처리 

  ```python
  while True:
      element_page_navigation = soup.select('.page_navigation')
      element_pages = element_page_navigation.select('a')
    
      # list 보다는 set 으로 중복된 아이템 제거
      pages.update({int(e.text) for e in element_pages if e and e.text.isdigit()})
    
      if next_page in pages:
          # 다음 페이지가 존재할때...
          continue
      else:
          break
  ```

### Telegram 또는 iMessage 메시지 보내기

  - Telegram

    발급 받은 TELEGRAM_BOT_TOKEN 과 chat id 로 메시지 보내기 

    ```python
    def send_telegram_message(message):
        url = f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage"
        payload = {
            "chat_id": TELEGRAM_CHAT_ID,
            "text": message
        }
        requests.post(url, json=payload)
    ```
    
  - iMessage
    macos 인 경우 osascript 를 이용해서 메시지 전송
    (단, csrutils 로 state 변경 필요)

    ```python
    def send_imessage(message):
        os.system(f"""osascript -e 'tell application "Messages" to send "{message}" to buddy "email or phone"'""")
    ```

        


    