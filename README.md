# Automate iOS Simulator with askui



## Demonstration

https://user-images.githubusercontent.com/115455389/215232967-c8f669bf-b3fe-4997-817c-9ad7e2a3a68b.mp4


## Overview

This article demonstrates how to use **askui** to automate web searching on iOS Simulator.

⚠️ Note that, for now, there is no support for automating real iOS devices. The method shown in this article is **only for iOS Simulator** running on **macOS**. 

In fact, **askui** has no official support for automation on iOS (including the Simulator) yet. But we can automate the iOS Simulator running on macOS, because **askui** performs the automation on the OS level, hence, we can possibly automate everything that is running within the operating system 😎


Taking it into account, let's start by checking the requirements.


## Requirements

- **[askui](https://docs.askui.com/docs/general/Getting%20Started/getting-started)**
    - askui is provided as an **npm package**. So you will need to prepare **Node.js** and install the required dependencies for askui. See [this tutorial](https://docs.askui.com/docs/general/Getting%20Started/getting-started) for a more descriptive tutorial for installing askui.
- **[Xcode](https://developer.apple.com/xcode/)**
    - **Xcode** ships together with several latest versions of **iOS Simulator**. So we don't need an extra step for installing the Simulator, but we need to install the **Xcode** which is provided in the App Store.
- **Xcode Command Line Tools**
    - If you have **Xcode** installed, but not the **Xcode Command Line Tools**, run this command in the terminal to install it:
        ```bash
        xcode-select --install
        ```


## 1. Install askui and Prepare the Test Code

- Clone this repository and install askui along with its dependencies:
```bash
git clone https://github.com/HaramChoi-askui/ios-simulator-automation
cd ios-simulator-automation
npm install
npx askui init
```

- Go to `test/helper/my-first-askui-test-suite.test.ts` and change the code as below:

```ts
import { aui } from './helper/jest.setup';

function ascriptTextInput(text: string) {
    return `osascript -e 'tell application "System Events"' -e 'keystroke "${text}"' -e 'end tell'`;
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

## 2. Breaking Down the Test Code

**1) Launch the iOS Simulator**
- This test step executes a shell command that launches the **iOS Simulator**.
- Note the `waitFor(5000)`. It waits for 5 seconds for the Simulator to fully launch, but it can possibly take more than 5 seconds depending on the processing power of your device.
```ts
it('should launch the Simulator', async () => {
        console.log("Launching the Simulator");
        await aui.execOnShell('open -a Simulator').exec();
        await aui.waitFor(5000).exec(); // wait for the simulator to launch
});
```

<br>

**2) Open the Safari Web Browser and Type the Keyword**
- Here we click on the small **Search** widget that pops up the **Spotlight** of the iOS. 
- Note that there is a small helper function `ascriptTextInput()` at the top of the test code. This function returns a string of an **Apple Script** that automates the keyboard input:

```ts
function ascriptTextInput(text: string) {
return `osascript -e 'tell application "System Events"' -e 'keystroke "${text}"' -e 'end tell'`;
}
```
- We type the name of the app we want to launch into the **Spotlight** by using a shell command.
- After launching the web browser, we click on the text "Search or enter website name" which is the placeholder of the search bar.

```ts
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
```

<br>

**3) Remove the Cookie Consent**
- If you are using a freshly installed Simulator, you will likely be asked to give consent for using cookies on every website.
- This code block examines whether you have faced a pop-up for the cookies, and will click on the **Reject all** button if there is any.

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
- Since we cannot perform mouse scrolling within the iOS Simulator, we push the **Arrow Down** key in order to perform the scrolling.
- The **Arrow Down** key can scroll just a tiny bit, so we perform it multiple times, in our case, 10 times. This must be adjusted depending on the screen size of your Simulator.

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


## Conclusion

Since the mobile app market gets bigger, app test automation is also becoming more important, especially testing the user interface could be the most critical factor that refines the user experience.

Although the **askui** has no official support for automating iOS devices yet, it still gives a huge benefit in automated tests performed on the iOS Simulator. Hang on tight, since there will be official support for iOS devices in the near future.

If you got issues while following this article, feel free to ask for help in our [Discord Community](https://discord.gg/KFYJ5xuyBA)!