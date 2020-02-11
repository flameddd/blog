# nvm slow down zsh launch

nvm 拖慢 zsh 啟動、超級慢  

找到 @belozyorcev 方案來處理
- https://github.com/nvm-sh/nvm/issues/539#issuecomment-403661578

## before `~/.bash_profile`
```bash
export NVM_DIR=~/.nvm
source ~/.nvm/nvm.sh
```

## after `~/.bash_profile`
```bash
# Install zsh-async if it’s not present
if [[ ! -a ~/.zsh-async ]]; then
  git clone https://github.com/mafredri/zsh-async.git ~/.zsh-async
fi
source ~/.zsh-async/async.zsh

export NVM_DIR="$HOME/.nvm"
function load_nvm() {
    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
    [ -s "$NVM_DIR/bash_completion" ] && . "$NVM_DIR/bash_completion"
}

# Initialize worker
async_start_worker nvm_worker -n
async_register_callback nvm_worker load_nvm
async_job nvm_worker sleep 0.1
```

## screen shot
![img](/assets/img/nvm_before.jpg)  
![img](/assets/img/async_nvm_after.jpg)  