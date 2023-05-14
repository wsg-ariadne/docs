<p align="center">
<img src="https://raw.githubusercontent.com/wsg-ariadne/ariadne/main/public/assets/logo.svg" width="200" alt="ariadne"><br><br>
<a href="https://chrome.google.com/webstore/detail/ariadne/dpnmlgmdfkpbbmjilppkbkhahmlckajg"><img src="https://storage.googleapis.com/web-dev-uploads/image/WlD8wC6g8khYWPJUsQceQkhXSlv1/iNEddTyWiMfLSwFD6qGq.png" width="150" alt="Available in the Chrome Web Store"></a>
</p>

# ariadne

![build-extension workflow](https://github.com/wsg-ariadne/ariadne/actions/workflows/build-extension.yml/badge.svg)

Ariadne is a browser extension that helps you become aware of [deceptive design](https://deceptive.design) in cookie banners on the Web. It does this through a set of specifically-trained machine learning models that detect deceptive design patterns within the language and the design of cookie banners.

It is compliant with Manifest V3, the new extension API for Google Chrome and Chromium-based browsers. It is also compatible with Firefox, for which it uses the older Manifest V2 API, since the V3 API is not yet fully supported [as of the time of writing.](https://bugzilla.mozilla.org/show_bug.cgi?id=1578284)

Ariadne is part of an ongoing research project on the automated detection of deceptive design in cookie banners.

## Installation

The latest builds of Ariadne are published on the [Releases page.](https://github.com/wsg-ariadne/ariadne/releases/latest)

### Firefox

- Download the **`ariadne-mv2`** ZIP file.
- Go to `about:debugging#/runtime/this-firefox` in Firefox.
- Click on `Load Temporary Add-on...` and select the ZIP file.

### Google Chrome, Chromium

Ariadne can be installed from the Chrome Web Store. Just click the Chrome Web Store badge at the top of this README :-)

Alternatively, you can install it manually:

- Download the **`ariadne-mv3`** ZIP file.
- Go to `chrome://extensions` in Chrome.
- Enable `Developer mode` in the top right corner.
- Drag and drop the ZIP file into the browser window.

The process should be similar for other Chromium-based browsers (Edge, Opera, Brave, etc.), just make sure to enable the developer mode in the extensions page before installing the extension.

Ariadne should now be installed. You can now visit any website and click on the extension icon to see stats for the current page.

## Development

### Prerequisites

You will need [Node.js](https://nodejs.org/en/) (v16 or higher). We recommend installing the latest LTS version.

Install the dependencies:

```bash
npm install
```

### Building

To build the extension **in development mode**, run:

```bash
# Firefox
npm run dev
# Chrome
npm run dev:v3
```

Vite will automatically rebuild the extension when you make changes to the source code. You will need to reload the extension in your browser to see the changes.

In development mode, the extension will attempt to contact [Dionysus](https://github.com/wsg-ariadne/dionysus) at `http://localhost:5000` for the data. Make sure to run Dionysus locally before running the extension in this mode. You can change this by setting the `VITE_API_URL` environment variable in the `.env.development` file.

To build the extension **in production mode**, run:

```bash
# Firefox
npm run build
# Chrome
npm run build:v3
```

In production mode, the extension will attempt to contact Dionysus at `https://ariadne.dantis.me`. You can change this by setting the `VITE_API_URL` environment variable in the `.env.production` file.

### Installing

The built extension files will be stored in `dist/` for Firefox and `dist-v3/` for Chrome. You can load the extension in your browser by following the instructions in the [Installation](#installation) section, but instead of loading the ZIP file,

- load the `manifest.json` file in `dist/` for Firefox, or
- click `Load unpacked` and select the `dist-v3/` directory for Chrome.

## Details on the Project

The project is divided into several repositories for different units of the project. Calliope and Janus are both models that are developed on Python, Dionysus is the backend, and Olympus provides a dashboard for [Ariadne](https://github.com/wsg-ariadne/ariadne). 

To view the repository and documentation for each of these, you can click on the header for each section below.

### [Calliope](https://github.com/wsg-ariadne/calliope)
📜 Calliope is the language clarity model for [Ariadne](https://github.com/wsg-ariadne/ariadne).

Janus is a Naive-Bayes classifier that is meant to process the text from a cookie banner and classify it into the following classes:

1. `GOOD` indicating that the language used in the extract is likely to be clear, descriptive, and provides options to provide or deny cookie consent
2. `BAD` indicating that the language used in the extract is likely to be confusing, vague, and assuming that cookie consent will be provided

This classifier allows [Ariadne](https://github.com/wsg-ariadne/ariadne) to determine whether a website uses deceptive design in the form of unclear language on its cookie banner.

### [Janus](https://github.com/wsg-ariadne/janus)
⚖️ Janus is the option weight model for [Ariadne](https://github.com/wsg-ariadne/ariadne).

Janus is an image classifier based on the VGG-19 model that classifies images into the following classes:

1. `absent` indicating that the option to refuse cookies is not on the interface at all
2. `weighted` indicating that the option to refuse cookies is made less obvious, less visible, or more tedious to select than the option to accept it
3. `even` indicating that the options to accept and refuse cookies appear on the cookie banner and are equally obvious.

This classifier allows [Ariadne](https://github.com/wsg-ariadne/ariadne) to determine whether a website uses deceptive design in the form of weighted options on its cookie banner.

### [Dionysus](https://github.com/wsg-ariadne/dionysus)
📚 Dionysus is the backend for the Ariadne project. It is made up of three parts:

1. An instance of the [Calliope](https://github.com/wsg-ariadne/calliope) model, which classifies cookie banner text as 'good' (i.e., clear enough, likely not deceptive) or 'bad' (i.e., intent is not clear, likely deceptive).
2. An instance of the [Janus](https://github.com/wsg-ariadne/janus) model, which classifies options or checkboxes in a cookie banner screencapture as 'absent' (no options detected), 'even' (evenly-weighted options detected), or 'weighted' (unevenly-weighted options detected).
3. A connection to a PostgreSQL database that stores user-generated reports of deceptive design in webpages, along with a [REST API](#api) that allows for report management and model usage.

### [Olympus](https://github.com/wsg-ariadne/olympus)
🌋 Olympus is the web-based dashboard for the Ariadne project. It works with [Dionysus](https://github.com/wsg-ariadne/olympus) to display summary statistics of all the user-submitted reports of deceptive design, along with a preview of the most recent reports. It is powered by [Nuxt v3](https://nuxt.com/) and [Tailwind CSS](https://tailwindcss.com/).

Olympus also makes use of the [Auth0](https://auth0.com) SDK, which allows for login functionality. At the moment Olympus does not include functionality that necessitates login, but in the future this could allow administrators to delete reports on user request, allow registered users to verify, dispute, and comment on reports, etc., although this is not currently planned and is beyond the current scope of the Ariadne research project.
