"use strict"

//
// NPM Packages
//

// Masto.js
const { createRestAPIClient } = require ("masto");

// Imports enviromental variables from .env
// Used for tokens and configuration
require('dotenv').config();

// A sleeping function to ensure minimal API abuse
function sleep (ms) {
    return new Promise (resolve => setTimeout(resolve, ms || process.env.DEF_DELAY));
}

// Signs into Mastodon
const masto = createRestAPIClient({
    url: process.env.MASTO_URL,
    accessToken: process.env.MASTO_TOKEN,
});

// Announces which instance it's trying to sign into
console.log('Assigned to Mastodon instance @' + process.env.MASTO_URL);

// Creates an object to easily origanise and grab e926.net related variables without flooding scope with similar names
const e926 = {
    url: "https://e926.net",
    query: "/posts.json?&tags=eeveelution+feral&limit=100", // Currently set to pull the most recent 100 posts, tagged Eeveelution and feral
    username: process.env.E926_USERNAME,
    apiKey: process.env.E926_APIKEY,
};

async function e926Fetch () {
    // Perform a GET request from https://e926.net/posts
    let response = await fetch((e926.url + e926.query), {
        // Provides e926.net Basic Authorization via the given username and apiKey
        headers: { "Authorization": "Basic " + btoa(`${e926.username}:${e926.apiKey}`) }
    });

    console.log(response);
    return await response.json();
};

(async function loop() {
    console.log("Starting Fetch");

    // Holds the chosen post, variable reset upon loop
    let artPost;
    
    await e926Fetch().then((data) => {
        // If query was successful then randomly pick a post and assign to artPost
        artPost = data.posts[Math.floor(Math.random() * data.posts.length)];
        console.log(artPost);
    });

    // Hold for at least 2 seconds
    console.log("Delayed for 2 seconds");
    await sleep(2000);

    // Grabs the image from e926.net and returns the file as a blob
    const remoteFile = await fetch(artPost.file.url).then(r => r.blob());

    // Mastodon Support
    // Prepares the image for Masto.js
    const attachment = await masto.v2.media.create({

        file: remoteFile,
        // Alt Text
        description: `An artwork with an eeveelution created by "${artPost.tags.artist[0]}". The artwork's description from e926 is "${artPost.description}"`
    });

    // Concatinates the original post, and it's sources
    let artSources = () => {
        return `Created by: ${artPost.tags.artist[0]}
    
        ${e926.url}/posts/${artPost.id}
        Sources:
        ${artPost.sources.join("\n")}`
    };

    // Post to Mastodon
    const status = await masto.v1.statuses.create({
        status: artSources(),
        visibility: "public",
        mediaIds: [attachment.id]
    })
    
    // Outputs the post
    console.log(status);

    // After the loop has been run, start a hour long delay.
    setTimeout(loop, 3600000);
})(); // Runs the function upon initalisation