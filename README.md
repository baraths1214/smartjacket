# smartjacket
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <errno.h>
#include <stdlib.h>
#include <stdio.h>
#include <wiringPi.h>

#define  BUFSIZE  128
#define RelayPn 3
#define  fet 21
#define  Red 22
#define Yellow 23
#define Green 25
int main(void)
{
    float temp;
    int i, j;
        int fd;
    int ret;
        char buf[BUFSIZE]; 
        char tempBuf[5];
         if(wiringPiSetup() == -1)
         {
         printf("Wiring Pi Initialization Failed");
         return 1; 
         }

        pinMode(fet, OUTPUT);
    pinMode(Red, OUTPUT);
    pinMode(Yellow, OUTPUT);
    pinMode(Green, OUTPUT);
    analogWrite(fet, 255);
    pinMode(RelayPn, OUTPUT);

    while(1)
               {    
        fd = open("/sys/bus/w1/devices/28-03119779e442/w1_slave", O_RDONLY);

        if(-1 == fd)
                {
            perror("open device file error");
            return 1;
        }

        while(1)
                       {
            ret = read(fd, buf, BUFSIZE);
            if(0 == ret){
                break;    
            }
            if(-1 == ret){
                if(errno == EINTR){
                    continue;    
                }
                perror("read()");
                close(fd);
                return 1;
            }
        }

        for(i=0;i<sizeof(buf);i++){
            if(buf[i] == 't'){
                for(j=0;j<sizeof(tempBuf);j++){
                    tempBuf[j] = buf[i+2+j];     
                }
            }    
        }

        temp = (float)atoi(tempBuf) / 1000;

        printf("%.3f C\n",temp);

        close(fd);

if ( temp <=15 && temp >= 10)
{
   digitalWrite(Green, HIGH);
   digitalWrite(Yellow,LOW);
   digitalWrite(Red,LOW);
   digitalWrite(RelayPn,HIGH);
}
else if (temp <= 20 && temp >= 15)
{
   digitalWrite(Yellow, HIGH);
   digitalWrite(Green,LOW);
   digitalWrite(Red,LOW);
   digitalWrite(RelayPn,HIGH);
}
else if ( temp >= 20 )
{
digitalWrite(Red,HIGH);
digitalWrite(Yellow,LOW);
digitalWrite(Green,LOW);
digitalWrite(RelayPn,LOW);
}
else
digitalWrite(RelayPn, LOW);

    }

    return 0;
}
