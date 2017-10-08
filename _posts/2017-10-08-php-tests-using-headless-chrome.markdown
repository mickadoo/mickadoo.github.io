---
layout: post
title:  "PHPUnit Tests Using Headless Chrome"
date:   2017-10-08 23:40:00 +0100
categories: php,phpunit,chrome,testing
---

Since [Headless Chrome][headless-chrome] was announced I was wondering how I could write tests in PHP using it.

From a fairly unlikely source, a post on [the PHP subreddit][reddit-post] I found someone who had written a [mink driver for Chrome][mink-driver].

I'd only ever heard about Mink when used along with Behat, although after looking around a bit it seems like [not everyone][mink-behat-post] agrees that they should be used together.

I won't repeat anything from the readme for the driver, so just [follow it][driver-readme] to get set up.

Then it's up to you to decide how you want to write your tests. Here's one basic sample. 

After working usually with browser emulator style tests with Symfony it was nice to see things like `getScreenshot`

```php
<?php

use Behat\Mink\Session;
use DMore\ChromeDriver\ChromeDriver;

class WebTest extends PHPUnit_Framework_TestCase {

  /**
   * @var ChromeDriver
   */
  protected $driver;

  public function setUp() {
    $chromeUrl = 'http://localhost:9222';
    $this->driver = new ChromeDriver($chromeUrl, NULL, 'http://dmaster.dev');
  }

  /**
   * @test
   */
  public function testWelcomeTextIsShown() {
    $session = $this->startSession();
    $session->visit('http://dmaster.dev');
    file_put_contents(__DIR__ . '/screenshot.png', $session->getScreenshot());
    $page = $session->getPage();
    $this->assertTrue($page->hasContent('welcome'));
  }

  /**
   * @return Session
   */
  protected function startSession() {
    $session = new Session($this->driver);
    $session->start();

    return $session;
  }
}
```

[headless-chrome]: https://developers.google.com/web/updates/2017/04/headless-chrome
[reddit-post]: https://www.reddit.com/r/PHP/comments/6axmyu/chrome_headless_support_without_selenium_for_mink/
[mink-driver]: https://gitlab.com/DMore/chrome-mink-driver
[mink-behat-post]: http://elnur.pro/behat-and-mink-are-not-meant-to-be-together/
[driver-readme]: https://gitlab.com/DMore/chrome-mink-driver/blob/master/README.md
