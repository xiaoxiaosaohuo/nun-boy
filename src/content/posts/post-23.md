---
title: A complete guide on how to take full page screenshots with Puppeteer

date: 2023-07-31T22:13:05.274Z
description: ""
image: "images/posts/puppetter.png"
categories: ["Tech"]
authors: ["siven"]
tags: ["Tech"]
draft: false
---

## Take a full page screenshot of the Apple site

```
import {NextResponse } from "next/server";
import puppeteer, { Page } from 'puppeteer';
import { HostWhiteList } from '@/config';

type ImageType = "png" | "jpeg" | "webp" | undefined;

const maxScrollCount = 10; // 设置滚动次数的上限

async function scroll(page:Page) : Promise<void> {
  await page.evaluate(async () => {
    return await new Promise<void>((resolve, reject) => {
      let scrollCount = 0;
      const i = setInterval(() => {
        window.scrollBy(0, window.innerHeight);
        const scrollingElement = document.scrollingElement || document.documentElement;
        if (
        scrollingElement.scrollTop + window.innerHeight >=
        scrollingElement.scrollHeight
      ) {
      window.scrollTo(0, 0);
      clearInterval(i);
      resolve();
    }
      scrollCount++;
      if (scrollCount >= maxScrollCount) {
        clearInterval(i);
        resolve();
      }
    }, 100);
  });
  });

}

export async function GET(
    request: Request,
  {
    params,
  }: {
    params: { code: string };
  }
  ) {
  const url = new URL(request.url);

  const { searchParams } = url;
  const browser = await puppeteer.launch({headless: "new"});
  try {
    const pageUrl = searchParams.get('url') || '';
    const pageUrlObj = new URL(pageUrl);
      if (!HostWhiteList.includes(pageUrlObj.host)) {
        return NextResponse.json(
          {
            success: false,
            msg: 'Invalid request'
          },
          { status: 400 }
        );
      }
      const width = Number(searchParams.get('width')) || 1080;
      const height = Number(searchParams.get('height')) || 1024;
      const deviceScaleFactor = Number(searchParams.get('ratio')) || 1;
      const type = (searchParams.get('type') || 'png') as ImageType ;
      const page = await browser.newPage();
      await page.setViewport({ width, height,deviceScaleFactor });
      await page.goto(
        pageUrl,
        { waitUntil: ['load', 'domcontentloaded'] }
      );
      await scroll(page);
      const imageRes = await page.screenshot({ type, fullPage: true });
      return new NextResponse(imageRes, {
            headers: {
              "Content-Type": `image/${type}`,
              "cache-control":"public, immutable, no-transform, max-age=31536000"
            },
          });

  } catch (e:any) {
  return NextResponse.json(
    {
      success: false,
      msg: e.message
    },
    { status: 500 }
  );
  } finally {
    await browser.close();
  }

  }

```

## Take screenshot by sections and then merge them together

What if a page have animations, the animated section only display when you scroll.

```
const puppeteer = require('puppeteer');
const merge = require('merge-img');
const Jimp = require('jimp');

async function scrollDown(page) {
    return await page.evaluate(() => {
        window.scrollBy(0, window.innerHeight);

        return window.scrollY >= document.documentElement.scrollHeight - window.innerHeight;
    });
};

function wait(milliseconds) {
    return new Promise((resolve) => {
        setTimeout(resolve, milliseconds);
    });
}

(async () => {
    const browser = await puppeteer.launch({ headless: true });
    try {
        const page = await browser.newPage();

        await page.goto(`https://apitoolkit.io/`);

        const path = 'screenshot.png';

        const { pages, extraHeight, viewport } = await page.evaluate(() => {
            window.scrollTo(0, 0);
            const pageHeight = document.documentElement.scrollHeight;
            return {
                pages: Math.ceil(pageHeight / window.innerHeight),
                extraHeight: (pageHeight % window.innerHeight) * window.devicePixelRatio,
                viewport: {
                    height: window.innerHeight * window.devicePixelRatio,
                    width: window.innerWidth * window.devicePixelRatio,
                },
            };
        });

        const sectionScreenshots = [];
        for (let index = 0; index < pages; index += 1) {
            // wait until animations are played
            await wait(400);

            const screenshot = await page.screenshot({ type: 'png', captureBeyondViewport: false });
            sectionScreenshots.push(screenshot);

            await scrollDown(page);
        }

        if (pages === 1) {
            const screenshot = await Jimp.read(sectionScreenshots[0]);
            screenshot.write(path);

            return screenshot;
        }

        if (extraHeight > 0) {
            const cropped = await Jimp.read(sectionScreenshots.pop())
                .then((image) => image.crop(0, viewport.height - extraHeight, viewport.width, extraHeight))
                .then((image) => image.getBufferAsync(Jimp.AUTO));

            sectionScreenshots.push(cropped);
        }
        const result = await merge(sectionScreenshots, { direction: true });

        await new Promise((resolve) => {
            result.write(path, () => {
                resolve();
            });
        });
    } catch (e) {
        console.log(e)
    } finally {
        await browser.close();
    }
})();
```

## References

[A complete guide on how to take full page screenshots with Puppeteer ](https://screenshotone.com/blog/a-complete-guide-on-how-to-take-full-page-screenshots-with-puppeteer-playwright-or-selenium/#fixing-the-most-issues-at-once)
