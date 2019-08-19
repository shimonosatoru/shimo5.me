+++
title = "Contact"
date = "2014-04-09"
sidemenu = "true"
description = "How to contact me"
+++

<form class="pure-form pure-form-stacked" name="contact" method="POST" data-netlify="true">
  <fieldset>
    <div class="pure-g">
      <div class="pure-u-1 pure-u-md-1-3">
        <label for="name">Name</label>
        <input id="name" class="pure-u-23-24" type="text" name="name" required>
      </div>

      <div class="pure-u-1 pure-u-md-1-3">
        <label for="email">E-Mail</label>
        <input id="email" class="pure-u-23-24" type="email" name="email" required>
      </div>
    </div>
    <fieldset class="pure-group">
      <input type="text" class="pure-u-23-24" placeholder="A title" name="title">
      <textarea class="pure-u-23-24" placeholder="Your message" name="message" required></textarea>
    </fieldset>
    <button type="submit" class="pure-button pure-button-primary">Send</button>
  </fieldset>
</form>