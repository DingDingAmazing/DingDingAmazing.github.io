---
layout: blog
istop: true
newest: true
winapp: true
title: "windows service for auto-clean"
background-image: https://picstore-1256648361.cos.ap-shanghai.myqcloud.com/windows_service/service.jpg
date:  2018-01-23 17:08
category: windows service
tags:
- windows service
---

# .net_C#  windows Service auto-clean specified folder
## Requirement
创建一个windows service，该service每隔30分钟检查一次指定的文件夹，清除里面存放超过2天的文件，并将文件操作记录在*.txt中。
## Realization
 -  新建C# windows service project
 - Service1.cs代码
 - 在Service1.cs [Design]中右键Add Istaller
 - 在ProjectInstaller.cs [Design]中分别设置：
**serviceInstaller1 properties：**
Description（系统服务的描述）
DisplayName （系统服务中显示的名称）
ServiceName（系统事件查看器里的应用程序事件中来源名称）
**serviceProcessInstaller1属性properties：**
Account -> LocalSystem
 - Project properties page Startup object选择*.program
 - 目标端在VS使用的相同的.net Framework版本的目录，复制release的service exe到该目录
 - 使用管理员权限的命令行安装service：
`InstallUtil Service1.exe`
 
 - 如果要卸载该service，使用管理员权限的命令行输入：
 `InstallUtil -u Service1.exe`

**Service1.cs代码:**
```C#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Diagnostics;
using System.Linq;
using System.ServiceProcess;
using System.Text;
using System.Threading.Tasks;
using System.IO;

namespace FailLogAutoClean
{
    public partial class Service1 : ServiceBase
    {
        System.Timers.Timer timer1;
        public Service1()
        {
            InitializeComponent();
            if(!System.Diagnostics.EventLog.SourceExists("2.0FailLogClean"))
            {
                System.Diagnostics.EventLog.CreateEventSource("2.0FailLogClean", "DailyCleanLog");
            }
            eventLog1.Source = "2.0FailLogClean";
            eventLog1.Log = "DailyCleanLog";
        }

        protected override void OnStart(string[] args)
        {
            eventLog1.WriteEntry("Sevice has started!");
            timer1 = new System.Timers.Timer();
            timer1.Interval = 60*60*30;//30minutes to elapse
            timer1.Elapsed += new System.Timers.ElapsedEventHandler(timer1_Elapsed);
            timer1.AutoReset = true;
            timer1.Enabled = true;
            eventLog1.WriteEntry("Real-time-clean has been running!");

            writestr("Service starts!");
        }

        protected override void OnStop()
        {
            eventLog1.WriteEntry("Sevice has stoped");
            timer1.Enabled = false;

            writestr("Service stops!");
        }

        private void timer1_Elapsed(object sender, System.Timers.ElapsedEventArgs e)
        {
            eventLog1.WriteEntry("Start real-time clean");
            writestr("Autoclean 04 Script Execute Report");
            try
            {
                Path01Clean();
                Path02Clean();
            }
            catch(Exception err)
            {
                writestr(err.Message);
            }
            writestr_01("\r\n");
        }

        public void Path01Clean()
        {
            try
            {
                //folder 01 path
                string strFolderPath01 = @"D:\IT&V Automation Test\ICI 2.0 Script\02 K226\04 Script Execute Report\Fail Image Folder";
                DirectoryInfo dyInfo01 = new DirectoryInfo(strFolderPath01);
                //get all files in the path
                foreach (FileInfo feInfo01 in dyInfo01.GetFiles())
                {
                    //delete all files 2 days before today
                    if(feInfo01.CreationTime.Year < DateTime.Today.Year)
                    {
                        feInfo01.Delete();
                        writestr_01("Delete" + feInfo01.Name + feInfo01.Extension);
                    }
                    else
                    {
                        if (feInfo01.CreationTime.DayOfYear < DateTime.Today.DayOfYear - 2)
                        {
                            feInfo01.Delete();
                            writestr_01("Delete" + feInfo01.Name + feInfo01.Extension);
                        } 
                    }
                }
             }
            catch(Exception err)
            {
                writestr(err.Message);
            }
        }

        public void Path02Clean()
        {
            try
            {
                //folder 02 path
                string strFolderPath02 = @"D:\IT&V Automation Test\ICI 2.0 Script\02 K226\04 Script Execute Report";
                DirectoryInfo dyInfo02 = new DirectoryInfo(strFolderPath02);
                foreach (FileInfo feInfo02 in dyInfo02.GetFiles())
                {
                    if (feInfo02.CreationTime.Year < DateTime.Today.Year)
                    {
                        feInfo02.Delete();
                        writestr_01("Delete" + feInfo02.Name + feInfo02.Extension);
                    }
                    else
                    {
                        if (feInfo02.CreationTime.DayOfYear < DateTime.Today.DayOfYear - 2)
                        {
                            feInfo02.Delete();
                            writestr_01("Delete" + feInfo02.Name + feInfo02.Extension);
                        }
                    }
                }
            }
            catch (Exception err)
            {
                writestr(err.Message);
            }
        }

        public void writestr(string readme)
        {
            StreamWriter dout = new StreamWriter(@"D:/CustomService/" + System.DateTime.Today.ToString("yyyyMMdd") + ".txt", true);
            dout.Write("\r\n" + System.DateTime.Now.ToString("yyy-MM-dd HH:mm:ss") + "    " + readme);
            dout.Close();
        }

        public void writestr_01(string readme)
        {
            StreamWriter dout = new StreamWriter(@"D:/CustomService/" + System.DateTime.Today.ToString("yyyyMMdd") + ".txt", true);
            dout.Write("\r\n" + readme);
            dout.Close();
        }
    }
}

```