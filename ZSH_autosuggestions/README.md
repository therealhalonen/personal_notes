# Setting up ZSH -autosuggestions and -syntax-highlighting   
*I have enjoyed the "autofill/suggestion" settings in Kali terminal (zsh), so i wanted to have them in my daily systems too*   

### Setup:   
```bash
# Debian 11
~$ uname -rsv
Linux 5.10.0-20-amd64 #1 SMP Debian 5.10.158-2 (2022-12-13)
```
```bash
# Centos 9 Stream
[~]$ uname -rsvn 
Linux centos9 5.14.0-252.el9.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Feb 1 13:25:18 UTC 2023
```
### Sources i used for this thing:    
[https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md "https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md")       [https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md "https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md")

### For Debian 11:    
```bash
sudo apt install zsh zsh-syntax-highlighting zsh-autosuggestions
# Run zsh to configure necessary stuffs to your liking
zsh

# Enable necessary, previously installed stuff to ZSH sessions:
echo "source $(dpkg -L zsh-autosuggestions | grep 'zsh$')" | tee -a ~/.zshrc
echo "source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" | tee -a ~/.zshrc

# Restart zsh session
source ~/.zshrc
```

### For Centos 9 Stream:   
```bash
# Add repos and install packages
sudo dnf config-manager --add-repo https://download.opensuse.org/repositories/shells:zsh-users:zsh-syntax-highlighting/CentOS_8_Stream/shells:zsh-users:zsh-syntax-highlighting.repo
sudo dnf config-manager --add-repo https://download.opensuse.org/repositories/shells:zsh-users:zsh-autosuggestions/CentOS_8_Stream/shells:zsh-users:zsh-autosuggestions.repo
sudo dnf install zsh-syntax-highlighting zsh-autosuggestions

# Enable ZSH history
echo "HISTFILE=~/.zsh_history" | tee -a ~/.zshrc
echo "HISTSIZE=1000" | tee -a ~/.zshrc
echo "SAVEHIST=1000" | tee -a ~/.zshrc

# Enable necessary, previously installed stuff to ZSH sessions:
echo "source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh" | tee -a ~/.zshrc
echo "source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" | tee -a 
# Restart ZSH session
source ~/.zshrc
```

