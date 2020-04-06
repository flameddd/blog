# Mac 上 menubar Progress Time

1. visit https://github.com/matryer/bitbar
2. goto release page, download zip
3. unzip, move to `Application` folder
4. create a folder (e.x. BitBarPlugin)
5. open `bitbar` (double click), set plugin folder (step4)
6. visit https://getbitbar.com/plugins/Time/progress.1h.sh
7. click `Add to BitBar`

（樣子稍微不一樣，是因為我自己改了一下程式碼）  
![BitBar_progress_time](/assets/img/BitBar_progress_time.jpg)  

line 52
```sh
echo "$(round "$Y_progress")%/$(round "$m_progress")%/$(round "$d_progress")%"
```