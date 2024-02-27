# Aprtc python following github of google
1. branch code information
    - master_old: nhánh code cũ, đang chạy cho version aar và chrome index: 1.0.27771
    - master: nhánh update liên tục từ github google `https://github.com/webrtc/apprtc`
    - main: nhánh main nhưng có modify code sao cho vẫn merge được từ github google `master`
2. install requirements
```bash
npm -v
# 6.14.17
# node version:
nvm use v14.16.0
# update newest code from github to "main" branch
git checkout main
## update code from google: https://github.com/webrtc/apprtc
git pull google master
# push to my github: git@github.com:uav-project-com/apprtc.git
git push origin main
## update nodejs lib
npm install
## pip version
python -m pip --version
# pip 20.3.4 from /usr/local/lib/python2.7/dist-packages/pip (python 2.7)
# update python lib
pip install -r requirements.txt
## install grunt
sudo apt install node-grunt-cli

### https://viblo.asia/p/tim-hieu-ve-grunt-6BAMYkkBvnjz
#### BUILD ######
grunt build

# output is: `out/app_engine`
```

3. fix bug build
- grunt build error:
    + https://github.com/gruntjs/grunt/issues/1737#issuecomment-1050182219
    ```bash
        nvm install v14.16.1
        npm install -g grunt-cli
        npm install grunt
    ```
    ran `sudo npm i`
in grunt-cli installed directory (/usr/share/nodejs/grunt-cli)


4. Deploy
- Copy `out/app_engine` into:
`/media/huyen/Data/admasterlife-webrtc-final-1.2021/docker-openssh-server/apprtc-image/webrtc/`
- overwrite constants.py and build docker file following hieu instruction
