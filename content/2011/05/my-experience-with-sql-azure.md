Title: My Experience with SQL Azure
Date: 2011-05-09 01:09
Author: Surya
Category: Rant
Slug: my-experience-with-sql-azure
Status: published

We are in the process of developing a prototype for Windows Azure.  
Since its a fairly a new technology not many online help/discussions  
are available. and if there are, then they are obsolete (as these steps
would become in couple of months)  
the reason being, Microsoft comes out with updates for major/minor
versions pretty quickly.  
Anyway these are the challenges we faced until now and maybe would face
some more in future. Most of them are  
due to our ignorance or lack of knowledge on how it could be done in a
certain way (thats one more problem with new tech, where  
nobody knows what\`s the standard way until you make errors).  
Now Azure requires latest and greatest. Hence, we decided to use virtual
machine to make Win 7 images and install azure sdk  
and sql server 2008 (big mistake for sql server...as we painfully found
out later that we needed SQL server 2008 R2 CTP\!).  
So during R\&D, we realized that we could use Windows Azure with Storage
as our backend. But, it turned out to be a  
Hash Table, which is only used to save key value pair, where value can
be anything from blob to complex objects etc. However again, we needed a
RDBMS. On further research, we found a service called SQL Azure which
was exactly what we required for Storage.  
And now when we tried to open it with Management Studio, we could only
do scripts and not objects.. Bummer\!\! I am more  
used to the GUI aspect of sqlserver then scripts, which made us search
for different alternatives in codeplex i.e.  
http://hanssens.org/tools/sqlazuremanager/ which were all buggy or
mostly incomplete\! Turns out we had to install  
SQL server 2008 R2 CTP. sigh\!\!\! Who knew??  
Once everything was set, we now had to move our existing data from
oracle to Azure. We used Microsoft's tool \<???\> to migrate data, which
was slow but worked fine. Finally, it was a success...Yippee\!\!
