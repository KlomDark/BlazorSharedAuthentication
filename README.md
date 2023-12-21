# DN8MultiLogin

Project to demonstrate passing auth info between multiple Blazor apps

## How to run the demo

This demo contains two separate Blazor sites that need to run at the same time. The solution needs to be set up to run both. To do that, use the following steps:

Note: If you're using VS Code, then you will have to edit the launch profile, and I am not sure what to put in it to run two at a time. Sorry, I don't use VS Code much. Please let me know if you figure it out, I'd like to know.

- First, create a C:\inetpub\keyring directory. .NET will use this directory to store some required key data it uses while running the app. (You can delete that directory once you are done with this demo.) Note that if you use this solution in production, it should be protected by encryption. This version is not encrypted. See [Encrypt Data Protection Keys At Rest](https://learn.microsoft.com/en-us/aspnet/core/security/cookie-sharing?view=aspnetcore-8.0#encrypt-data-protection-keys-at-rest) for information on how to do this.
- Open the solution in Visual Studio.
- In Solution Explorer, right-click on the Solution itself at the top.
- Select Properties.
- In the Startup Project tab, select the Multiple Startup Project radio button.
- Set Primary and Secondary to Start.
- Leave Primary.Client and Secondary.Client set to None.
- Hit OK.
- Press F5 to run the solution.
- Two browsers will open. Find the one that says Primary in the top left. Ignore Secondary, it has to be running but we're going to ignore that browser. Don't hit it yet, but we will be switching to the Secondary app via the button in Primary that is on the main page.
- First, click the Register tab. (This solution is set up to use LocalDB for storing authentication for simplicity. If you're on Linux you'll have to update the connection strings in both the Primary and Secondary projects to point to a real database server, since LocalDB only runs on Windows.)
- Fill in your info (It can be a fake email, it is just used within this app.)
- The first time you run the app, you'll get an error the first time you try to register an account. In the error message will be an Apply Migrations button. Click it to create the authentication database. Refresh the page and say Yes to the question about resubmitting the form data.
- In the 'Register confirmation' page that appears, click the link that says 'Click here to confirm your account'.
- Now click on the 'Auth Required' tab and log in using the account you just created.
- You will now see "You are authenticated" followed by your email address.
- OK, now you are logged in to the Primary app.
- Now click on the 'Home' tab
- Note that the top-left says "Primary" to indicate that you are still in the Primary app.
- Now we are going to transfer the active login over to the Secondary app by clicking the Secondary button on the main page.
- First, notice that the top left now says "Secondary" to indicate we have switched over to the Secondary app. (You might also notice that the port number in the URL has changed to a different number.)
- Now click on the 'Auth Required' tab.
- This time, instead of having to log in, it will say "You are authenticated" followed by your email address.
- You have successfully transferred between Blazor apps using the same credentials!
- End of demo

## What makes this possible?

If you open the Program.cs file in both the Primary and Secondary projects, you will see the same identical code that's been added. This sets up both apps to use the same name for the Authentication cookie.

The added code:
```
builder.Services.AddAuthentication(options =>
    {
        options.DefaultScheme = IdentityConstants.ApplicationScheme;
        options.DefaultSignInScheme = IdentityConstants.ExternalScheme;
    })
    .AddIdentityCookies();

builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"C:\inetpub\keyring"))
    .SetApplicationName("DemoSharedCookie");

builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.Name = ".Demo.SharedCookie";
});
```