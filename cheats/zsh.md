# ZSH

## Install

Install Zsh.

```bash
brew install zsh
```

Install Oh My Zsh.

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

Upgrade it.

```bash
upgrade_oh_my_zsh
```

Change theme

Edit the config file.

```bash
nano ~/.zshrc
```

### References

[https://github.com/robbyrussell/oh-my-zsh/issues/1906\#issuecomment-252443982](https://github.com/robbyrussell/oh-my-zsh/issues/1906#issuecomment-252443982)

[https://www.freecodecamp.org/news/how-to-configure-your-macos-terminal-with-zsh-like-a-pro-c0ab3f3c1156/](https://www.freecodecamp.org/news/how-to-configure-your-macos-terminal-with-zsh-like-a-pro-c0ab3f3c1156/)

## Change theme.

```text
#ZSH_THEME="robbyrussell"
ZSH_THEME="agnoster"
```

Reload the config.

```bash
source .zshrc
```

### References

[https://github.com/robbyrussell/oh-my-zsh/wiki/Themes](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes)

## Install fonts

Run.

```text
git clone https://github.com/powerline/fonts.git
cd fonts
./install.sh
```

Change your iTermr2 font to Meslo LG L for Powerline.

## Fix autocomplete

Put these two lines at the end of my `.zshrc`

```bash
autoload -Uz compinit
compinit
```

## Shorten path

Edit your theme confi file.

```bash
cd ˜/.oh-my-zsh/themes/
vi agnoster.zsh-theme
```

Edit the file as follows.

```bash
prompt_dir() {
  #prompt_segment blue $CURRENT_FG '%~'
  prompt_segment blue $CURRENT_FG '%2~'
}
```

Apply changes.

```text
source ˜/.zshrc
```

### Update ZSH after the change

```text
cd ~/.oh-my-zsh
git stash
upgrade_oh_my_zsh
git stash pop
```

### 

