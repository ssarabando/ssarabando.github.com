---
layout: post
title: Don't use RsaProtectedConfigurationProvider with ProtectSection
---

h1. {{ page.title }}

p(meta). 6 Aug 2012

This one cost me a couple of hours last week.
I have an install helper that all of my installation projects use that allows changing application settings and connection strings in a graphical UI instead of mucking around in the configuration files.
Up to this point, all that wasn't being encrypted as the product was still on a "beta" stage and it was practical to have it all in plain text.
Now that the product is ready to be sent into production, the need to encrypt some sections in the configuration files arose.
I decided to use the @ProtectSection@ method of the @SectionInformation@ class as it seemed the most cost efficient way of doing it.
Since it expects one of two possible "protection providers" (@RsaProtectedConfigurationProvider@ or @DataProtectionConfigurationProvider@), and not being an expert in security issues, I googled around a little in order to choose one.
As most of the articles that I found about the subject focused on the RSA provider (and I did not found any clear advantage in using the other one) I decided to go with that.
In order to implement the encryption I only needed to change the save method's logic which ended up somewhat like this:

{% highlight c# %}
private void UpdateConnectionString(ConnectionStringsSection csSection,
    IEnumerable<KeyValuePair<string, string>> connStrings)
{
    foreach (var connString in connStrings)
    {
        var key = connString.Key;
        var matchingConnString = csSection.ConnectionStrings
            .Cast()
            .SingleOrDefault(cs =>
                StringComparer.OrdinalIgnoreCase.Equals(cs.Name, key));
        if (matchingConnString != null)
            matchingConnString.ConnectionString = connString.Value;
    }
    csSection.SectionInformation.ProtectSection("RsaProtectedConfigurationProvider");
    csSection.SectionInformation.ForceSave = true;
}
{% endhighlight %}

This worked (as in no errors thrown and the section was encrypted) but when I tried to read it back, it would throw an _Unrecognized element 'setting'_ exception.
After fiddling around with the code and doing quite a few "googles", I found out that it was a bug in the .NET 4 Framework with more than one year of age.
According to "this":http://support.microsoft.com/kb/2548766 page at Microsoft Support, this happens because _«white space is not preserved by the method in the *RSACryptoServiceProvider* class.»_
The only way to solve this is by either installing an hotfix (which you can get only by contacting Microsoft Support) or using another provider.
Since I have no desire of having to deploy an hotfix to every customer and development/testing machine, I ended up using the @DataProtectionConfigurationProvider@ instead.

So, in summary: don't use the @RsaProtectedConfigurationProvider@ with @ProtectSection@. It'll bite you.
