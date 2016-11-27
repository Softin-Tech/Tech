# 合并多个commit

在feature分支上，可能有一些临时的commit，而这些commit可能共同解决一个小的问题，避免造成commit历史的混乱，我们可能需要合并这些commit

如图，有3次commit，我们要将second commit 和 third commit 合并成一次commit，并更新message

`$ git log`

<img height='300px' src='../public/img/Screen Shot 2016-11-24 at 4.50.28 PM.jpg'>

<br>
`$ git rebase -i cd98edc81b14946d26273e5c3f85b8c667ab1262`

把third commit的command改成squash

> squash 或 fixup：这次commit会被合并到前面的commit（两者区别在于commit message是否保留）

<img height='350px' src='../public/img/Screen Shot 2016-11-24 at 5.05.54 PM.jpg'>

<br>
`:wq 保存退出vi`

还有机会修改commit message

<img height='300px' src='../public/img/Screen Shot 2016-11-24 at 5.12.46 PM.jpg'>

修改message为"bug fixes"

<img height='200px' src='../public/img/Screen Shot 2016-11-24 at 5.16.10 PM.jpg'>

`:wq 保存退出vi`

Successfully rebased and updated refs/heads/master.

##### 注意
rebase操作可能有危险，如果中途出现任何错误，不用担心，因为此时并不是在原来的分支上操作，而是在 *(no branch, rebasing master) 分支上，通过git rebase --abort可以撤销合并修改
