# git 相关

##  杂项

1. **git status不显示中文**
    
		git config --global core.quotepath false
				
2. **git clone 出现Setting locale failed.**
	
		perl: warning: Setting locale failed.
		perl: warning: Please check that your locale settings:
		LANGUAGE = (unset),
		LC_ALL = (unset),
		LC_CTYPE = "zh_CN.UTF-8",
		LANG = "en_US.UTF-8"
   		are supported and installed on your system.
		perl: warning: Falling back to the standard locale ("C").
		
	解决方案：
	
		OSX
		
		添加一下内容到 ~/.bash_profile
		
		export LC_CTYPE=en_US.UTF-8
		export LC_ALL=en_US.UTF-8
		
		如果使用的是 zsh, 添加一下内容到 ~/.zshrc
		
		LC_CTYPE=en_US.UTF-8
		LC_ALL=en_US.UTF-8
		
3. **修改最近一次提交的内容**

		git commit --amend




