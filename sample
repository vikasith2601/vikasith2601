#include &lt;winsock2.h&gt;
#include &lt;stdio.h&gt;
#include &lt;assert.h&gt;
#include &quot;SocketHelper.h&quot;
//If you have an older version of winsock2.h
#ifndef SO_EXCLUSIVEADDRUSE
#define SO_EXCLUSIVEADDRUSE ((int)(~SO_REUSEADDR))
#endif
/*
  This application demonstrates a generic UDP-based server.
  It listens on port 8391. If you have something running there,
  change the port number and remember to change the client too.
*/
int main(int argc, char* argv[])
{
    SOCKET sock;
    sockaddr_in sin;
    DWORD packets;
    bool hijack = false;
    bool nohijack = false;
    if(argc &lt; 2 || argc &gt; 3)
    {
        printf(&quot;Usage is %s [address to bind]\n&quot;, argv[0]);
        printf(&quot;Options are:\n\t-hijack\n\t-nohijack\n&quot;);
        return -1;
    }
    if(argc == 3)
    {
        //Check to see whether hijacking mode or no-hijack mode is 
        //enabled.
        if(strcmp(&quot;-hijack&quot;, argv[2]) == 0)
        {
            hijack = true;
        }
        else
        if(strcmp(&quot;-nohijack&quot;, argv[2]) == 0)
        {
            nohijack = true;
        }
        else
        {
            printf(&quot;Unrecognized argument %s\n&quot;, argv[2]);
            return -1;

        }
    }
    if(!InitWinsock())
        return -1;
    //Create your socket.
    sock = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if(sock == INVALID_SOCKET)
    {
        printf(&quot;Cannot create socket -
  err = %d\n&quot;, GetLastError());
        return -1;
    }
    //Now let’s bind the socket.
    //First initialize the sockaddr_in.
    //I’m picking a somewhat random port that shouldn’t have 
    //anything running.
    if(!InitSockAddr(&amp;sin, argv[1], 8391))
    {
        printf(&quot;Can’t initialize sockaddr_in - doh!\n&quot;);
        closesocket(sock);
        return -1;
    }

    //Let’s demonstrate the hijacking and 
    //anti-hijacking options here.
    if(hijack)
    {
        BOOL val = TRUE;
        if(setsockopt(sock, 
                      SOL_SOCKET, 
                      SO_REUSEADDR, 
                      (char*)&amp;val, 
                      sizeof(val)) == 0)
        {
            printf(&quot;SO_REUSEADDR enabled -  Yo Ho Ho\n&quot;);
        }
        else
        {
            printf(&quot;Cannot set SO_REUSEADDR -  err = %d\n&quot;, 
                   GetLastError());
            closesocket(sock);
            return -1;
        }
    }

    else
    if(nohijack)
    {
        BOOL val = TRUE;
        if(setsockopt(sock, 
                      SOL_SOCKET, 
                      SO_EXCLUSIVEADDRUSE, 
                      (char*)&amp;val, 
                      sizeof(val)) == 0)
        {
            printf(&quot;SO_EXCLUSIVEADDRUSE enabled\n&quot;);
            printf(&quot;No hijackers allowed!\n&quot;);
        }
        else
        {
            printf(&quot;Cannot set SO_ EXCLUSIVEADDRUSE -  err = %d\n&quot;, 
                   GetLastError());
            closesocket(sock);
            return -1;
        }
    }
    if(bind(sock, (sockaddr*)&amp;sin, sizeof(sockaddr_in))  == 0)
    {
        printf(&quot;Socket bound to %s\n&quot;, argv[1]);
    }
    else
    {
        if(hijack)
        {
            printf(&quot;Curses! Our evil warez are foiled!\ n&quot;);
        }
        printf(&quot;Cannot bind socket -  err = %d\n&quot;, GetLastError());
        closesocket(sock);
        return -1;
    }
    // OK, now we’ve got a socket bound. Let’s see whether someone
    //sends us any packets - put a limit so that we don’t have to 
    //write special shutdown code.
    for(packets = 0; packets &lt; 10; packets++)
    {
        char buf[512];
        sockaddr_in from;
        int fromlen = sizeof(sockaddr_in);
        // Remember that this function has a TRINARY return;

        //if it is greater than 0, we have some data;
        //if it is 0, there was a graceful shutdown 
        //(shouldn’t apply here);
        //if it is less than 0, there is an error.
        if(recvfrom(sock, buf, 512, 0, (sockaddr*)&amp;from, &amp;fromlen)&gt; 
0)
        {
            printf(&quot;Message from %s at port %d:\n%s\n&quot;,
                   inet_ntoa(from.sin_addr),
                   ntohs(from.sin_port),
                   buf);
            // If we’re hijacking them, change the message and
            //send it to the real server.
            if(hijack)
            {
                sockaddr_in local;
                if(InitSockAddr(&amp;local, &quot;127.0.0.1&quot;, 83 91))
                {
                    buf[sizeof(buf)-1] = ’\0’;
                    strncpy(buf, &quot;You are hacked!&quot;, siz eof(buf) -
1);
                    if(sendto(sock, 
                              buf, 
                              strlen(buf) + 1, 0, 
                              (sockaddr*)&amp;local, 
                              sizeof(sockaddr_in)) &lt; 1)
                    {
                        printf
                     (&quot;Cannot send message to localhost -
 err = %d\n&quot;,
                      GetLastError());
                    }
                }
            }
        }
        else
        {
            
            printf(&quot;Ghastly error %d\n&quot;, GetLastError() );
            break;
        }
    }
    return 0;
}
