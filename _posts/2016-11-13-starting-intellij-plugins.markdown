---
layout: post
title:  "Creating a 'Hello, World!' plugin for Intellij"
date:   2016-11-13 18:40:00 +0100
categories: phpstorm,intellij
---

I started out with a new company that uses the Phalcon framework, which I'd never used before.
Opening up the code in my IDE I was hit by countless warnings because the IDE didn't know about.

Coming from a Symfony background where the PHPStorm plugin tells the IDE all it needs to know for 
code completion it was a big change. I really liked being able to leave all the pre-commit checks on and still be able to
commit with no warnings. As of writing this there is no PHPStorm plugin for Phalcon that I could find.

I've no experience with Java so I wasn't expecting it to be easy. The [intellij docs][1] are not bad and [this video][5] was helpful, 
but I really just wanted a simple step-by-step guide to getting set up.

## Steps

- Make sure you have the [Java JDK][3] installed
- Follow the [instructions here][4] to download the source code of the community edition
- Download the [Intellij Community Edition][2]
- Unzip it and move it to wherever you want
- Run it from the directory you moved it to with `./bin/idea.sh`
- Create a new Java project with any settings. The guide tells you to set up an SDK but without a project I couldn't do this
- Go to File > Project Structure
- Under Platform > SDKs choose to add a new one
- You'll get a popup asking you to choose the home directory for the Intellij Plugin SDK. Choose the default which should be where you moved the Intellij Community
Edition you're using right now
- Choose to create a JDK if none exist. Point to the directory where the JDK (from step one) was installed. In my case it was /usr/lib/jvm/java-8-openjdk-amd64
- Click on the "Sourcepath" tab. Add the directory of the source code you cloned in step two
- Give your configuration a name and save it
- Choose "New Project" from the file menu and select "Intellij Platform Plugin". Chose the SDK you just created from the dropdown list.
- Right clicking on the src folder I chose New > Action
- For id I chose "sayHelloWorld", for class 'helloWorldAction', for name "Hello World" and whatever for description
- You need to choose a group to say where the action is accessible from. I chose "GenerateGroup" and the "last" radio button but it could be any menu you like.
- This generated a new class which had a method `actionPerformed`. It also automatically updated `resources/META-INF/plugin.xml`
- Inside the `actionPerformedMethod` I added the code below.
- If the run configurations on the top right are empty create a new one for "Plugin" accepting the defaults
- Run the configuration and a new instance of Intellij should start
- Inside this instance create a new project or open an existing one. From inside the editor go to Code > Generate.. and you should see your new action
- Choose it and you'll see "hello world" in the middle of the screen. Fantastic eh!?

### helloWorldAction class

```
package com.mickadoo.helloWorld;

import com.intellij.openapi.actionSystem.AnAction;
import com.intellij.openapi.actionSystem.AnActionEvent;
import com.intellij.openapi.ui.popup.JBPopupFactory;

public class helloWorldAction extends AnAction {

    @Override
    public void actionPerformed(AnActionEvent e) {
        JBPopupFactory factory = JBPopupFactory.getInstance();
        factory.createMessage("Hello, World!").showInFocusCenter();
    }
}
```

[1]: http://www.jetbrains.org/intellij/sdk/docs/basics/getting_started.html
[2]: https://www.jetbrains.com/idea/download/#section=linux
[3]: http://openjdk.java.net/install/
[4]: http://www.jetbrains.org/intellij/sdk/docs/tutorials/custom_language_support/prerequisites.html
[5]: https://www.youtube.com/watch?v=-ZmQD6Fr6KE
