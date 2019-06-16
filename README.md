import os
import time
import psutil
import urllib.request
import smtplib
import schedule
import threading
from sys import *
from email import encoders
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
from email.mime.multipart import MIMEMultipart


def is_connected():
    try:       
        urllib.request.urlopen('http://216.58.192.142',timeout = 1)
        return True
    except Exception:
        return False

def MailSender(filename,time):
    try:
        fromaddr = "shwetashruti1996@gmail.com"
        toaddr = "shrutidesai.903@gmail.com"
 
        msg = MIMEMultipart()
        msg['From'] = fromaddr
        msg['To'] = toaddr
        body = """
        Hiiiiiii %s,
        These is send by shweta desai through automation script mail sender program..
        Log file is created at : %s


        This is auto generated mail.

        Thanks & Regards,
        Shweta Desai
        """%(toaddr, time)

        Subject = """
        Process log generated at : %s
        """%(time)

        msg['Subject'] = Subject
        msg.attach(MIMEText(body,'plain'))
        attachment = open("/Users/LENOVO/AppData/Local/Programs/Python/Python37-32/log_path.txt", "rb")
        p = MIMEBase('application','octat-stream')
        p.set_payload((attachment).read())
        encoders.encode_base64(p)
        p.add_header('Content-Disposition',"attachment; filename= %s" % filename)
        msg.attach(p)

        s = smtplib.SMTP('smtp.gmail.com', 587)
        s.starttls()
        s.login(fromaddr, "--------password here--------")
        text = msg.as_string()
        s.sendmail(fromaddr, toaddr, text)
        s.quit()
        print("Log file successfully send through mail")

    except Exception as E:
        print("Unable to send mail",E)

def ProcessLog(log_path = 'log_path'):
    listprocess = []
    
    if not os.path.exists(log_path):
        try:
            os.mkdir(log_path)
        except:
            pass

    seprator = "-" * 80
    
    f = open('log_path','w')
    f.write(seprator + "\n")
    f.write("Process logger : "+time.ctime()+"\n")
    f.write(seprator + "\n")
    f.write("\n")
    
    for proc in psutil.process_iter():
        try:
            pinfo = proc.as_dict(attrs=['pid','name','username'])
            vms = proc.memory_info().vms/(1024*1024)
            pinfo['vms'] = vms
            listprocess.append(pinfo)
            
        except(psutil.NoSuchProcess, psutil.AccessDenied, psutil.ZombieProcess):
            pass

    for element in listprocess:
        f.write("%s\n" %element)


    print("Log file is generated at location %s"%(log_path))

    connected = is_connected()

    if connected:
        startTime = time.time()
        MailSender(log_path, time.ctime())
        endTime = time.time()

        print('Took  %s seconds to execute'%(endTime - startTime))
        print("Execution completed at "+time.ctime())

    else:
        print("Their is no internet connection")

def counter():
    schedule.every(int(1)).minutes.do(ProcessLog)
    while True:
            schedule.run_pending()
            time.sleep(1)
        


def main():
    print("Application Name : "+argv[0])

    if(len(argv)==2):
        if (argv[1]=="-h")or(argv[1]=="-H"):
            print("Help - This script is designed for traversing through folder.")
            exit()

        if (argv[1]=="-u")or(argv[1]=="-U"):
            print("Usage - Provide Application Time Interval in minutes")
            exit()

    try:
        print("Execution started at "+time.ctime())
        t1 = threading.Thread(target = counter, args =())
        t1.start()
        t1.join()
    except ValueError:
            print("Error : Invaid datatype of input.")
    except Exception as E:
            print("Error : Invalid input.",E)
        

    
if __name__ == "__main__":
    main()

        


