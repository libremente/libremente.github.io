---
layout:     post
title:      "How to handle Open edX XBlocks localization"
date:       2017-12-06 00:00:00
author:     "surF"
header-img: "img/code-bg.jpg"
---

Open edX is the platform which powers MOOC websites like
[edx.org](http://edx.org), [Stanford's Lagunita](https://lagunita.stanford.edu)
and many other MOOC providers worldwide. 
Since it has been released with a FOSS license, its adoption has wildly
increased and right now millions of users worldwide are using such a platform
for their online courses. 
As such, the possibility to translate the interface should be a top priority
but, unfortunately, not all the components have such a functionality out of the
box. In fact, the platform's external modules, also called ``XBlocks``, are
difficult to translate and this article is meant to depict all the possible
ways to approach such a task.

Localization is handled pretty well in the *edx-platform*: there are dedicated
tools
to handle all the [translation
processes](https://github.com/edx/i18n-tools) and teams on Transifex
which help
translating the interfaces and the contents [[you may join them
here]](https://www.transifex.com/open-edx/edx-platform/). 

However, for what concerns the external modules which can be integrated in the
platform, called [XBlocks](https://open.edx.org/xblocks), there are still
issues to be faced and there
are ongoing efforts by the edX engineers in order to solve them. 
The existing integrated translation solution is not working at the moment of writing
(if you are lucky you may see the bug [here](https://openedx.atlassian.net/browse/WL-230)). 

Some XBlocks are correctly translated like, e.g., [ORA2](https://github.com/edx/edx-ora2) or edX Proctoring.
However, they are just isolated cases and, unfortunately, not all the XBlocks
developers follow the translations guidelines. 

In order to understand the nature of this problem it is necessary to understand
what is the nature of an XBlock inside the Open edX structure. At the moment of
writing, XBlocks are not *Django Apps* but they can be seen as standalone
modules. 
In
fact, XBlocks should be coded in an independent way with respect to the
edx-platform and it could also be possible to run them as standalone. Originally
called XModules, XBlocks should use standard Python and Mako template engine
exactly for the up-cited purpose. 
Furthermore, for
development purposes, it is possible to use a standalone software development
kit which does not require a running instance of Open edX installed ([see
here](https://edx.readthedocs.io/projects/xblock-tutorial/en/latest/getting_started/setup_sdk.html))
and makes it handy to develop new functionalities.
This means that the developers have to follow some strict guidelines regarding
the implementation of such blocks like, e.g., the templating engine used, and this
is causing some issues with the interoperability with the existing localization
engine. 

For the sake of this quick guide, let's first of all analyze the way Django handles
translations.
First of all, all the message files containing the translations in a given language,
characterized by the `.po` extensions, are compiled into the machine readable 
`.mo` binary files. 
This usually in Django is done by using the 
```
django-admin makemessages -a
```
but in Open edX there are several other ways to do this, i.e. using the
i18n suite mentioned before. 
Afterwards, these message files have to be seen by Django in order to be
correctly applied.
The order of the search is the following:


1. the directories listed in `LOCALE_PATHS` have the highest precedence, with
   the ones appearing first having higher precedence than the ones appearing
   later.
2. Then, it looks for and uses if it exists a locale directory in each of the
   installed apps listed in `INSTALLED_APPS`. The ones appearing first have
   higher precedence than the ones appearing later.
3. Finally, the Django-provided base translation in `django/conf/locale` is used
   as a fallback.

So, analyzing the list we can see how number 2. is not our case since declaring
an XBlock as a Django App is not a best practice. I have tried this and it
works without problems but many developers advised me that it's not an
encouraged 
practice, even if a UI Architect at edX told me that it's not
a problem (check out the thread
[here](https://groups.google.com/forum/#!topic/openedx-translation/cLR5tZI5oqQ)).
However, it seems that in the futures XBlocks will be proper Django apps and
this can be seen in this OEP
[here](https://open-edx-proposals.readthedocs.io/en/latest/oep-0012.html#refactor-xblocks-to-be-based-upon-django).

Also, point number 3. is not so straightforward since it would mean that for
each new XBlock installed we ought to move the translation files to that
specific django folder, to create a new `.po` file and finally a `.mo` one. Not very handy.

Solution number 1. might a good pick since, by explicitly telling Django
where to look at, it should be able to pick the new translations. 

I tried to implement the different solutions and they are all depicted in [this
commit](https://github.com/libremente/edx-platform/commit/22292d5e26207eeb772778990e2f28196581030f).

The following piece of code adds solution number 1. to the `lms/startup.py`
file which is invoked during startup. 

```python
def enable_locale_discovery():
    """ 
    Enables Django to see and apply the translations in the XBlocks
    After retrying all the xblocks currently installed, it checks whether a
    `translations` folder exists and adds it to the LOCALE_PATHS list. 
    """
    import inspect, os
    from xblock.core import XBlock

    # Folder name where the XBlocks' translations are located
    locale_folder = 'translations'
    # Retry list of loaded XBlocks
    xblocks_list = XBlock.load_classes()
    for name, class_  in xblocks_list:
        # For each XBlock, get the absolute path of the compiled file 
        xblock_install_path = inspect.getfile(class_)
        # Paths have a recurrent form:
        # `/edx/app/edxapp/<install_path>/<xblock>/<xblock>/xblock.pyc`
        # Strip first '/edx/app/edxapp/' and last 'xblock.pyc' from the path
        stripped_path = xblock_install_path.split('/edx/app/edxapp/',1)[1].rsplit('/',1)[0]
        # Build path using ENV_ROOT
        translated_url = settings.ENV_ROOT / stripped_path / locale_folder
        # Check if the folder exists and if it is not empty
        if(os.path.isdir(translated_url) and os.listdir(translated_url)):
            # Check for unicity and then add to LOCALE_PATHS
            if(translated_url not in settings.LOCALE_PATHS):
                settings.LOCALE_PATHS = ( translated_url , ) + settings.LOCALE_PATHS
```

So in the way depicted above it is possible to add each XBlock's translation
folder to the `LOCALE_PATHS` read by Django and the translations are applied
consequently.

Another solutions implemented by [*Felipe
Montoya*](https://github.com/felipemontoya) implies the definition and the
usage of an extra Django app called `django-xblock-i18n`. This app basically
defines a new service which has to be loaded inside the context of the XBlock
and then it can be used. Furthermore, the `xblock_i18n` tag has to be added at
the top of the template tag to be used. For more extended info check the repo
I forked which contains some more information about how to use the app in the
README file [here](https://github.com/libremente/django-xblock-i18n).

The possibility of merging the `.po` files together is always possible even if
it is highly unrecommended due to all the possible problems that may occur
especially during upload - i.e. all the work would be lost after each upgrade.
However, what follows is the procedure to accomplish such a task.
For each XBlock to be localized, apply the following passages:

* extract the strings to be localized from all the files like, e.g., `.py` and
  `.js` ones. For `python`:

  ```bash
  $ find . -name "*.py" | xargs xgettext --language=python --add-comments="Translators:"
  ```

  For `javascript`:
  
  ```bash
  $ find . -name "*.js" -o  -path  ./public/js/vendor -prune -a -type f | xargs xgettext --language=javascript
  --from-code=utf-8 --add-comments="Translators:"
  ```

  Note that each command generates a `message.po` file, so after running the
  first time make sure to `mv` the file to another name. 

  NB: there is the possibility of adding `--join-existing` to the second
  command but, at the moment of writing, it is not working on my machine.
  That should help appending the
  second output to the first `message.po` file. 

  Anyway, Django expects to have a `django.{po,mo}` file in its locale
  directory which means that those `messages.{po,mo}` files have to be renamed. 

  For our case, all the strings extracted from `python` should exist in the
  `django.po` file, whilst the ones related to `javascript` should be inserted
  inside the `djangojs.po` file.
  
  To check if `django.po` is correct, you can run `msgfmt` to build
  a `django.mo` file:
  ```bash
  $ msgfmt django.po -o django.mo 
  ```

  If everything is correct, the resulting `django.mo` file has to be move
  to the final directory:
  `translations/<lang_code>/LC_MESSAGES/django.mo`.
  Same for the `djangojs.{po,mo}` files.

* Now that the `.po` file is generated, it is necessary to attach it to the
  platform's one located in `/conf/lang/<lang_code>/django.po`. When merging
  the two files, make sure that there are no conflicting strings (strings which
  are present in both files). If there are, remove the ones present in the new
  `django.po` file and leave the originals intact. Once the merge is finished,
  it is possible to run the command to generate the new `django.mo` file as
  before:

  ```bash
  $ msgfmt conf/locale/<lang_code>/LC_MESSAGES/django.po -o translations/<lang_code>/LC_MESSAGES/django.mo 
  ```

From now on, Django will read from those files and apply the translations
globally. However, as cited above, this is **not recommended** for obvious
reasons. 

So, in order to conclude, we have explored the various paths that Django checks
to spot the `.mo` files containing the translations to apply.
Three solutions have been proposed, which are more of a *workaround* than real
solutions but they may work while waiting for the official solution. 
Solution number 1. is what I consider being the best one since it's not as
*invasive* as the others: solution number 2. adds a Django App to the system
whilst 3. is a big mess since it basically implies that all the translation
will be merged in a big centralized file which, after an update or a change,
will be changed and will lose the mods. 

Let me know what you think and if what I wrote may be deemed acceptable.

That's all folks! :)







