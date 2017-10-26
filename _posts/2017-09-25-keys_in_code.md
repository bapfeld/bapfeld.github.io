---
layout: post
title: "Keys and Secret Data in Code"
categories: [Code]
---
So here's a problem that even those of us in the social sciences encounter occasionally: you have code to run that requires some kind of secret information, such as an API key. And inevitably at some point you're going to have to share this code --- either with a collaborator or for replication or maybe you've actually written something that others will find useful in the future.[^1] If you're sharing your code, you really don't want to share your secrets. While you might not be held financially liable in the end, you probably don't want the headache of having somebody run up a [$10,000 tab on AWS](https://wptavern.com/ryan-hellyers-aws-nightmare-leaked-access-keys-result-in-a-6000-bill-overnight) or get your account permissions revoked on some database crucial to your research.

The solution to this problem is relatively straightforward, but finding that solution is a rabbit hole of strong opinions argued using lots of programming knowledge I, for one, don't have. That said, here is the solution I've come to, laid out in what I hope are simple steps and explanations.

# The Wrong Way
Okay, so first up, let's briefly cover the *wrong* way to store your secrets, namely writing them directly into your code. Doing that leads to bad things.[^2] And writing them there while you run your code and then changing things before you share it won't work either (because you're using version control, right? right!?).

At risk of putting the cart before the horse, I'm also going to assert that putting the variables in *any* form *anywhere* in your project's directory is a mistake. That's how the coder in the linked story above ran into trouble.

So Rule 1 is NEVER write your secrets into code that you may eventually share; Rule 2 is NEVER write your secrets into any file that is in a directory under version control.

# My Preferred Solution
Store the variables in an encrypted file in a central location in your computer and then access that file, as needed, in your code. Better yet, use multiple files. My preferred file type is `.ini`. Others may disagree and say that json is better.[^3] Ini is often associated with Windows but despite this it's plenty portable and packages exist for its native inclusion in all the languages I write in. Plus it is ultimately a plain text file format and that means it can be used basically anywhere.

So what does this look like? Well you create a file called, for example, `my_secrets.ini` that contains:

```ini
[some_category]
my_var = my_value
```

You can have multiple categories (to ignore the multiple file suggestion, for example), comments, and some additional tools. You can also quote your values or include spaces (even unquoted ones) there. See [wikipedia](https://en.wikipedia.org/wiki/INI_file) for some more basic information. It's simple, readable, and easy to use.

Accessing those variables is quite easy. In R, for example, you can use the `ini` package to do this:

```R
my_secret_value <- ini::read.ini('/path/to/my_secrets.ini')$some_category$my_var
```

And in python, it's like this:

```python
import configparser
config = configparser.ConfigParser()
config.read('/path/to/my_secrets.ini')
my_secret_value = config['some_category']['my_var']
```

And so forth for any other language you might want to use.

# Security Concerns
Okay, so you're not publishing your secrets, but they're still saved in a file somewhere and those can be stolen. Yes, that's true, but there are some limits here. If you're really concerned about local files being stolen, then one option would be to place those files into some kind of secure vault (that you then unlock before you run your code). Having a special flash drive for this kind of thing is a similar sort of option. There are some encryption options, too, but you'll limit your options for using those files. And all of this is beyond the scope of this post (and, frankly, my knowledge). The bottom line is that yes, files can be stolen, but you can manage those risks with some reasonable precautions.

I promised to keep this simple, but I also want to be transparent. There are actually two solutions to this problem: save them into a file somewhere in a universally readable format (such as the ini method recommended above) or save your keys as environment variables (maybe in a `.env` file but absolutely NOT in your `.bash_profile` or similar). There are some really good reasons to use the latter option.  Environment variables are language and system agnostic, for one. My biggest hesitation in using that option in that environment variables are available to any process that's running within the environment. In other words, if your variables exist for *your* code to call, they also exist somewhere where someone else's malicious code to steal (which is precisely why you don't save them to your global environment). Now this is starting to be truly paranoid thinking, since the way this works would imply that you're using some module/package/application that has code that does this without you or anybody else who uses that open source software noticing. That's pretty unlikely. But environment variables are no easier to work with than variables in an ini file and they still exist in *a* file so they have the same security issues and don't solve cross-system portability issues. Bottom line: I'm using .ini files until I'm convinced otherwise.

[^1]: Okay, so that last one is pretty unlikely, but we can all dream, right?
[^2]: Don't believe me? Don't take my word for it --- do a search for "AWS credentials stolen large bill" and you'll find people claiming to have had hackers run up bills *much* larger than $10k.
[^3]: If you're inclined to this opinion, check out [this](https://arp242.net/weblog/json_as_configuration_files-_please_dont) post.
