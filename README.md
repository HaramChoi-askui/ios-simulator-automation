# Automate iOS Simulator with askui

## Overview

This article covers how to use **askui Library** to automate web searching on iOS Simulator.

UI test automation helps to ensure that the app's UI is functioning correctly as expected, which, in turn, improves the overall user experience and the app's chance of success in the market.

Since the UI Automation on Android devices has already been covered in [this article](https://www.askui.com/blog-posts/tutorial-automate-web-search-on-android-devices-with-askui), we will now cover how to perform **UI automation on iOS**.

Note that, for now, the method shown in this article is **only for iOS Simulator** running on macOS. 

In fact, **askui Library** has no official support for automation on iOS devices yet. But we can automate the iOS Simulator running on macOS, because **askui Library** performs the automation on the OS level, hence, we can possibly automate everything that is running within the operating system😎


Taking it into account, let's start by checking the requirements.

------

## Demonstration

https://user-images.githubusercontent.com/115455389/215232967-c8f669bf-b3fe-4997-817c-9ad7e2a3a68b.mp4


------

## Requirements

- **[askui](https://docs.askui.com/docs/general/Getting%20Started/getting-started)**
- **macOS**
- **[Xcode](https://developer.apple.com/xcode/)**
- **Xcode Command Line Tools**


If you are missing something mentioned above, then go to the respective link and install it.

-------

## 1. Prepare the askui Test Code

- Go to your askui code file and use the following code:

```ts
import { aui } from './helper/jest.setup';

function ascriptTextInput(text: string) {
    return `osascript -e 'tell application "System Events"' -e 'set txt to "${text}"' -e 'keystroke txt' -e 'end tell'`;
}

describe('iOS Simulator', () => {
    it('should launch the Simulator', async () => {
        console.log("Launching the Simulator");
        await aui.execOnShell('open -a Simulator').exec();
        await aui.waitFor(5000).exec(); // wait for the simulator to launch
    });
    it('should open safari', async () => {
        console.log("Opening Safari");
        await aui.click().text().withText('Search').exec();
        await aui.execOnShell(ascriptTextInput('safari')).exec();
        await aui.pressKey('enter').exec();
        await aui.waitFor(1000).exec(); // wait for the app to load
    });

    it('should search the keyword', async () => {
        console.log("Searching the keyword");
        await aui.click().text().withText("Search or enter website name").exec();
        await aui.execOnShell(ascriptTextInput('askui')).exec();
        await aui.pressKey('enter').exec();
        await aui.waitFor(2000).exec(); // wait for the page to load
    });

    it('should remove the cookie popup', async () => {
        console.log("Removing the cookie popup");
        try {
            await aui.expect().text().containsText('cookie').exists().exec();
            await aui.click().text().withText('Read more').exec();
            await aui.click().text().withText('Read more').exec();
            await aui.click().text().withText('Reject all').exec();
        } catch (error) {
            console.log('No cookie popup found. Moving on...');
        }
    });

    it('scroll down', async () => {
        console.log("Scrolling down");
        for(let i=0; i<10; i++) {
            await aui.pressKey('down').exec();
        }
    });

    it('should click on the link', async () => {
        console.log("Clicking on the link");
        await aui.click().text().withText('Installing askui').exec();
        await aui.waitFor(2000).exec(); // wait for the page to load
    });

    it('should remove the cookie popup', async () => {
        console.log("Removing the cookie popup");
        try {
            await aui.expect().text().containsText('cookie').exists().exec();
            await aui.click().text().withText('Allow selection').exec();
        } catch (error) {
            console.log('No cookie popup found. Moving on...');
        }
    });
});

```

------

## 2. Breaking Down the Test Code

**1) Launching the iOS Simulator**
- This test step executes a shell command that launches the iOS Simulator.
- The iOS Simulator must have been installed together with **Xcode**.
- Note the `waitFor(5000)`. It waits for 5 seconds for the Simulator to fully launch, and can possibly take more than 5 seconds depending on the processing power of your device.
```ts
it('should launch the Simulator', async () => {
        console.log("Launching the Simulator");
        await aui.execOnShell('open -a Simulator').exec();
        await aui.waitFor(5000).exec(); // wait for the simulator to launch
});
```

<br>

**2) Open the Safari Web Browser and Search the Keyword**
- Here we click on the small **Search** widget that popups the **Spotlight** of the iOS. 
- We type the name of the app we want to launch into the **Spotlight** by using a shell command.
- Note that there is a small helper function `ascriptTextInput()` at the top of the test code.
- This function returns a string that contains an Apple Script that performs the keyboard input for us.

```ts
function ascriptTextInput(text: string) {
return `osascript -e 'tell application "System Events"' -e 'set txt to "${text}"' -e 'keystroke txt' -e 'end tell'`;
}
```

<br>

**3) Remove the Cookie Consent**
- If you are using a freshly installed Simulator, it's likely that you will be asked to give consent for using cookies on every website.
- This code block examines whether you have faced a pop-up for the cookies, and will click on the **Reject All** button if there is any.

```ts
it('should remove the cookie popup', async () => {
console.log("Removing the cookie popup");
    try {
        await aui.expect().text().containsText('cookie').exists().exec();
        await aui.click().text().withText('Read more').exec();
        await aui.click().text().withText('Read more').exec();
        await aui.click().text().withText('Reject all').exec();
    } catch (error) {
        console.log('No cookie popup found. Moving on...');
    }
});
```

<br>

**4) Scroll Down the Search Result**
- Since we cannot use the scroll of the mouse within the iOS Simulator, we push the **Arrow Down** key in order to perform the scrolling.
- The **Arrow Down** key can scroll just a tiny bit, so we perform it multiple times, in our case, 10 times. This must be adjusted depending on the screen size of the Simulator.

```ts
it('scroll down', async () => {
    console.log("Scrolling down");
    for(let i=0; i<10; i++) {
        await aui.pressKey('down').exec();
    }
});
```

<br>

**5) Click and Go to the Target Website**
- After we finish scrolling as much as needed, we click on the desired link.
- Thereafter, we, again, check the cookie consent.

```ts
it('should click on the link', async () => {
    console.log("Clicking on the link");
    await aui.click().text().withText('Installing askui').exec();
    await aui.waitFor(2000).exec(); // wait for the page to load
});

it('should remove the cookie popup', async () => {
    console.log("Removing the cookie popup");
    try {
        await aui.expect().text().containsText('cookie').exists().exec();
        await aui.click().text().withText('Allow selection').exec();
    } catch (error) {
        console.log('No cookie popup found. Moving on...');
    }
});
```

And now we are done with our automation! 🎉

------

## Conclusion

Since the mobile app market gets bigger, app test automation is also becoming more important, especially testing the user interface could be the most critical factor that refines the user experience.

Although the **askui Library** has no official support for automating iOS devices yet, it still gives a huge benefit in automated tests performed on the iOS Simulator. Hang on tight, since there will be official support for iOS devices in the near future.

If you got issues while following this article, feel free to ask for help in our [Discord Community](https://discord.gg/KFYJ5xuyBA)!