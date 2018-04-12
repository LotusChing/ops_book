# lua基础语法

简单记了下常用的
```
[root@bj-vmware-test1 ~]# cat test.lua 
#!/usr/bin/lua
--print('Yo')
print("### Variables ###")
name = "LotusChing"
print(name)

-- For
print("### For ###")
sum = 0
for i = 1, 3 do
     print(i)
end

-- While
print("### While ###")
num = 3
while num > 0 do
    print(num)
    num = num -1
end

-- if
print("### IF ###")
my_age = 18
if my_age < 18 then
    print("Kid.")
elseif my_age == 18 then
    print("Boy.")
elseif my_age > 18 then
    print("Man.") 
end

current_user_role = "guest"
if current_user_role ~= "admin" then
    print("!!!Permission Deny!!!")
end
```

执行结果
```
[root@bj-vmware-test1 ~]# lua test.lua 
### Variables ###
LotusChing
### For ###
1
2
3
### While ###
3
2
1
### IF ###
Boy.
!!!Permission Deny!!!
```

## lua 教程
* [Lua简明教程](http://coolshell.cn/articles/10739.html)
* [lua在线lua学习教程](http://book.luaer.cn/)
* [Lua 5.1 参考手册](http://www.codingnow.com/2000/download/lua_manual.html)
* [Lua5.3 参考手册](http://cloudwu.github.io/lua53doc/)



