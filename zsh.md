# zsh 插件安装好了后, 如何启用.

在用户目录下的 .zshrc  文件中 添加如下内容:

source /usr/local/Cellar/zsh-autosuggestions/0.3.3/share/zsh-autosuggestions/zsh-autosuggestions.zsh

source /usr/local/Cellar/zsh-syntax-highlighting/0.5.0/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

fpath=(/usr/local/Cellar/zsh-completions/0.22.0/share/zsh-completions $fpath)
compinit



保存退出,然后执行 source .zshrc, 即可.

