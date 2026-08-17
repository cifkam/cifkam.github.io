---
layout: page
permalink: /contact/
title: contact
nav: true
nav_order: 5
description:
---

<form id="contact-form" class="contact-form">
  <input type="hidden" name="access_key" value="{{ site.web3forms_access_key }}">
  <input type="hidden" name="subject" value="New message from cifkam.github.io">
  <input type="hidden" name="from_name" value="{{ site.first_name }} {{ site.last_name }} — website">

  <!-- Honeypot: bots fill this in, humans never see it. -->
  <input type="checkbox" name="botcheck" class="contact-honeypot" tabindex="-1" autocomplete="off">

  <div class="form-group">
    <label for="contact-name">Name</label>
    <input type="text" class="form-control" id="contact-name" name="name" required>
  </div>

  <div class="form-group">
    <label for="contact-email">Your email</label>
    <input type="email" class="form-control" id="contact-email" name="email" required>
  </div>

  <div class="form-group">
    <label for="contact-message">Message</label>
    <textarea class="form-control" id="contact-message" name="message" rows="7" required></textarea>
  </div>

<button type="submit" class="btn btn-sm z-depth-0" id="contact-submit">Send</button>

  <div id="contact-result" class="contact-result" role="status" aria-live="polite"></div>
</form>

<style>
  .contact-form {
    max-width: 40rem;
  }
  .contact-form .form-group {
    margin-bottom: 1.25rem;
  }
  .contact-form label {
    font-weight: bold;
    margin-bottom: 0.25rem;
  }
  .contact-form .form-control {
    color: var(--global-text-color);
    background-color: #fff;
    border: 1px solid rgba(0, 0, 0, 0.2);
    border-radius: 4px;
    padding: 0.5rem 0.75rem;
    width: 100%;
    transition:
      border-color 0.2s ease,
      box-shadow 0.2s ease;
  }
  .contact-form .form-control:focus {
    background-color: #fff;
    border-color: var(--global-theme-color);
    box-shadow: 0 0 0 2px rgba(136, 51, 0, 0.12);
    outline: none;
  }
  .contact-form .form-control::placeholder {
    color: var(--global-text-color-light);
  }

  /* Matches the theme's own button style (publications page). */
  .contact-form .btn {
    background-color: #fff;
    color: var(--global-text-color);
    border: 1px solid var(--global-text-color);
    border-radius: 4px;
    padding: 0.35rem 1.5rem;
    box-shadow: none;
    transition:
      color 0.2s ease,
      border-color 0.2s ease;
  }
  .contact-form .btn:hover,
  .contact-form .btn:focus {
    background-color: #fff;
    color: var(--global-theme-color);
    border-color: var(--global-theme-color);
    box-shadow: none;
  }
  .contact-form .btn:disabled {
    background-color: #fff;
    color: var(--global-text-color-light);
    border-color: var(--global-divider-color);
    cursor: not-allowed;
  }
  .contact-honeypot {
    display: none !important;
  }
  .contact-result {
    margin-top: 1rem;
    min-height: 1.5rem;
  }
  .contact-result.error {
    color: var(--global-theme-color);
  }
</style>

<script>
  document.getElementById("contact-form").addEventListener("submit", function (event) {
    event.preventDefault();

    const form = event.target;
    const button = document.getElementById("contact-submit");
    const result = document.getElementById("contact-result");
    const originalLabel = button.innerHTML;

    button.disabled = true;
    button.innerHTML = "Sending…";
    result.className = "contact-result";
    result.innerHTML = "";

    fetch("https://api.web3forms.com/submit", {
      method: "POST",
      headers: { "Content-Type": "application/json", Accept: "application/json" },
      body: JSON.stringify(Object.fromEntries(new FormData(form))),
    })
      .then(async (response) => {
        const data = await response.json();
        if (response.ok) {
          result.innerHTML = "Thanks — your message has been sent.";
          form.reset();
        } else {
          result.className = "contact-result error";
          result.innerHTML = data.message || "Something went wrong. Please try again.";
        }
      })
      .catch(() => {
        result.className = "contact-result error";
        result.innerHTML = "Could not reach the mail service. Please try again later.";
      })
      .finally(() => {
        button.disabled = false;
        button.innerHTML = originalLabel;
      });
  });
</script>
