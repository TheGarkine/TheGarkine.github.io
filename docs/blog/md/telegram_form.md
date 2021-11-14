# Client-side Contact Forms with Telegram and Javascript

<img style="display: block; width: 100%;" src="/media/blogs/telegram_form/banner.webp" />

While migrating my website to a more light-weight (and more manageable) github-pages site, I came across the issue what to do with my contact form.
Before, I used wordpress which was using an smtp to send an email to my account.
Now I am restricted to providing static files and server-side code isn't an option any more.

While sending emails from the website via JS is possible, I had serious security concerns, and also thought the code I found on the matter rather bloated and unsatisfying.
Checking my current (and frequently used) contact methods, I decided that it would be nice to get the message via one of my instant messengers.
I used telegram in some projects before so I realized the capabilities, but before I was only sending messages from servers with no direct user interaction, so I had some concerns at first.

The two things I wanted to avoid were:
- My contact information getting leaked, resulting in an enormous amount of spam.
- The messages being accessible, I wanted users to be able to write personal messages without the fear of others reading them.

First of all, we need to understand what data is required to send a message.
Basically to have a bot contacting you via a program you need two pieces of information (aside from the message of course):
- The bot's API key
- Your chat message ID

I do not really care about about the bot's API key to be leaked.
Having a look at the [API definition](https://core.telegram.org/bots/api#available-methods), I realized that the bot cannot really cause any damage (except for me, and that just by spamming), when in the hands of others.
Note that I am not advising anyone to make their bots public, if your bot has more functionality and processes sensitive information you should keep your API key secure!
This scenario is very specific for a bot that is solely meant as a single-directional message provider.

The chat message ID isn't a security risk either.
Bots can only message you if you messaged them first so its fine if people know your ID.
At this point I want to emphazise on how much I like how Telegram bots are designed.
They are really thought through, so try them out in your projects if you have a similar use-case!

For the second problem of others reading private messages, we can refer again to a bots capabilities in the [API definition](https://core.telegram.org/bots/api#available-methods).
Basically there is no possible way for the bot to read specific messages, he can only handle ids and understand commands.
Since that is also dealt with I could start by creating a new bot for this project.

## Creating the Telegram Bot

Thanks to Telegram's awesome workflows and concepts, this is a very simple task, simply chat with [@BotFather](https://t.me/BotFather).
With the `/newbot` command you can create a new bot, give it a good username and you'll get an API key from the BotFather which we will use in a second.

<img style="display: block; margin-left: auto; margin-right: auto; width: 70%;" src="/media/blogs/telegram_form/BotFather.webp" />

## Making the Form

Well I am definitly no frontend guy, I know my HTML and CSS but thats it. So here is is my (very simple) bootstrap based contact form:

```html
<form id="contactForm">
    <div class="form-group">
        <label for="contactEmail">Email address</label>
        <input type="email" class="form-control" id="contactEmail" aria-describedby="emailHelp" placeholder="Enter email" required>
        <small class="form-text text-muted">I'll never share your email with anyone else.</small>
    </div>
    <div class="form-group">
        <label for="contactMessage">Message</label>
        <textarea class="form-control" id="contactMessage" rows="3" required></textarea>
    </div>
    <br>
    <button id="contactSubmit" type="submit" class="btn btn-primary">Submit</button>
</form>
```

It definitly does its job in existing as a form, that does not hurt your eyes too bad looking at it:

<img style="display: block; margin-left: auto; margin-right: auto; width: 90%;" src="/media/blogs/telegram_form/ContactForm.webp" />

## Coding some Javascript

Now let's do the magic; we have to write some javascript code in our frontend to actually send the message:

```js
const email = document.getElementById("contactEmail").value;
const message = document.getElementById("contactMessage").value;
const text = encodeURIComponent(`Email: ${email}\n\n---Message below---\n${message}`);

const url = `https://api.telegram.org/bot${botApiKey}/sendMessage?chat_id=${chatID}&text=${text}`;
const request = new XMLHttpRequest();
request.open( "GET", url, false ); // false for synchronous request
request.send();
```

Note the `encodeURIComponent` call on the message text.
Without you would use some characters like new-lines, which are not automatically encoded by `XMLHttpRequest`.
The `botApiKey` and `chatID` variables have to be set to the according values.

To give the user some feedback I disable the form interaction, and append a small message to say thank you:

```js
const form = document.getElementById("contactForm");

if (request.status === 200) {
    // Freeze the form
    document.getElementById("contactEmail").readOnly = true;
    document.getElementById("contactMessage").readOnly = true;
    document.getElementById("contactSubmit").disabled = true;

    // Append a thank you message
    const span = document.createElement("span");
    span.innerHTML = "Thank you for your message!"
    span.style.color="green";
    form.appendChild(span);
} else {
    // Append an error message
    const span = document.createElement("span");
    span.innerHTML = "An error occured please try again later!"
    span.style.color="red";
    form.appendChild(span);
}
```

The last thing required is putting the javascript in the submit event of the event.
Note that you should finally return `false` such that the form does not reload your page, and you are done.
If you want to peek into the full source code you can check the source code of the `index.html` file in this website's repository ([here](https://github.com/TheGarkine/TheGarkine.github.io)).

## Trying it out

So I sent an example message, with a few linebreaks and even a smilie; and instantly got this message in Telegram:

<img style="display: block; margin-left: auto; margin-right: auto; width: 50%;" src="/media/blogs/telegram_form/BotMessage.webp" />

Thank you for reading this blog!
I hope you learned something.
If you have any questions you know have a form to ask them!