gerrit
==========

ssh key
-----------

在gerrit上配置ssh key，邮箱。
 
初始化配置
--------------

    repo init -u https://xxxx  -b  zzzz
    repo init -u https://xxxx -b yyyy --no-repo-verify
    repo init -u https://xxxx -b yyyy -m xxx.xml --no-repo-verify
 
拉取代码
---------

    repo sync -j4 -c
    repo forall -c 'git lfs pull'

新建本地开发分支
------------------

    repo start --all myLocalDevBranch

查看分支
----------

    repo list
    repo overview
     
    cd .repo/manifests
    git branch -a
 
查看本地git配置
--------------------
 
    cat .git/config

    [branch "myLocalDevBranch"]
            remote = someRemoteName
            merge = refs/heads/SomeRemoteBranch


推送本地git commit到gerrit
------------------------------
 
    git push someRemoteName HEAD:refs/for/SomeRemoteBranch
 
在本地合并多个git commit
---------------------------

    git reset --soft HEAD~3
    git commit . -m xxxx 
