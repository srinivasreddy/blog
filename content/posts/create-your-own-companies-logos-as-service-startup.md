+++ 
draft = false
date = 2020-11-07T17:07:56+05:30
title = "Build companies logos as a service startup"
description = ""
slug = "" 
tags = ["startups","idea","sass"]
categories = []
externalLink = ""
series = []
+++


While working on a project, I have to write a program to crawl logo images of various companies. I have automated this with puppeteer script written in Typescript/Javascript. While this script has done the job for me - extracting 100 odd companies logos, I have to warn you that I haven't tested more than that.

You can install the required packages with the following commands,
```bash
npm i puppeteer puppeteer-extra-plugin-stealth --save
npm i reflect-metadata node-fetch fs puppeteer-extra --save
```
Then,
```typescript
import { Browser, BrowserContext } from "puppeteer";
import puppeteer, { PuppeteerExtra } from "puppeteer-extra";
import StealthPlugin from "puppeteer-extra-plugin-stealth";
import fetch from "node-fetch";
import fs from "fs";
    
async function startLogoDownloader() {
    const companyName = "Google";
    const jobs: Job[] = [];
    const inactiveUrls: string[] = [];
    puppeteer.use(StealthPlugin());
    const browser = await puppeteer.launch({ headless: false });
    const browserContext = await browser.createIncognitoBrowserContext();
    const page = await browserContext.newPage();
    const jobUrl = "https://google.com";
    try {
        await page.goto(jobUrl, { waitUntil: "networkidle2" });
        await page.type("input[type=text]", `facebook page of ${companyName} company`, { delay: 40 });
        await page.keyboard.press("Enter");
        await page.waitForSelector("div#search", { timeout: 10000 });
        await page.waitForSelector('a[href^="https://www.facebook.com"', { timeout: 10000 });
        const results = [];
        const urls = await page.evaluate((resultsObject) => {
            const urlResults = JSON.parse(resultsObject).results;
            document.getElementById("search").
            querySelectorAll('a[href^="https://www.facebook.com"').
            forEach((a: HTMLAnchorElement) => {
                urlResults.push(a.href);
            });
            return urlResults;
        }, JSON.stringify({ results }));

        if (urls.length === 0) {
            return;
        }
        // Take the first result
        await page.goto(urls[0], { waitUntil: "networkidle2" });
        await page.waitForSelector('a[aria-label="Profile picture"]', { timeout: 10000 });
        const imageUrl = await page.evaluate(() => {
            const element = document.querySelector('a[aria-label="Profile picture"]').querySelector("img");
            return element.src;
        });
        console.log(`The image url is ${imageUrl}`);
        const resp = await fetch(imageUrl);
        const buffer = await resp.buffer();
        const path = `./${companyName}.png`;
        fs.createWriteStream(path).write(buffer);
        await page.close();
        await browserContext.close();
        await browser.close();
} catch (e) {
        console.log(`Error(s) occured for ${companyName} : ${e.toString()}`);
    }
}
```
In file `logodownloader.ts`, you can invoke this script as,
```typescript
import "reflect-metadata";
import { startLogoDownloader } from "../common/logo_downloader";

(async () => {
    await startLogoDownloader();
})();
```
### TODO
1. You can generally store an image in an S3 bucket and store the path in the database or store it in DB as a blob. There are various pros and cons associated with each approach. Every few years this discussion comes up in DB mailing lists as `To blob or Not to blob` . You can google that expression. :)
### Key take aways from the script.
1. I have hard coded the company name - "Google" to demonstrate the example. You can pass an array of companies and run it.
2. Currently, there is no mechanism in the code to retry failed companies.
3. You can attach this to `npm run logo_download` by invoking the file `logodownloader.ts` in package.json.