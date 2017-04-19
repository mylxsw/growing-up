# 使用expect记住SSH登陆密码

使用`expect`命令，该命令语法采用了tcl编程语言
    
    #!/usr/bin/expect
    
    set timeout 20
    set ip "IP地址"
    set user "账号"
    set password "密码"
    
    spawn ssh "$user\@$ip"
    
    expect "$user@$ip's password:"
    send "$password\r"
    
    interact

# 使用SCP命令，自动输入密码

    #!/usr/bin/expect
    
    set timeout 3000
    
    set server_host "IP地址"
    set server_user "账号"
    set password "密码"
    
    set local_root "本地要上传的文件"
    set server_root "上传的目标目录"
    
    spawn scp -r $local_root $server_user@$server_host:$server_root
    
    expect {
        "*password:" {
            send "$password\r"
            exp_continue
        }
        "*?" {
            send "y\r"
        }
    }
    
    expect eof


# SCP循环使用，自动输入密码

    #!/usr/bin/expect
    
    set timeout 20
    set server_host "IP地址"
    set server_user "账号"
    set password "密码"
    
    set local_root "本地目录"
    set server_root "目标目录"
    
    while { [gets stdin line] >= 0 } {
        if {$line == "EOF"} {
            break
        }
    
        spawn scp -r $local_root$line $server_user@$server_host:$server_root$line
        expect "*password:"
        send "$password\r"
    
        expect eof
    }


