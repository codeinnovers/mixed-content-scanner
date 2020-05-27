# Scan your site for mixed content

[![Latest Version on Packagist](https://img.shields.io/packagist/v/spatie/mixed-content-scanner.svg?style=flat-square)](https://packagist.org/packages/spatie/mixed-content-scanner)
[![Build Status](https://img.shields.io/travis/spatie/mixed-content-scanner/master.svg?style=flat-square)](https://travis-ci.org/spatie/mixed-content-scanner)
[![SensioLabsInsight](https://img.shields.io/sensiolabs/i/7a85bc21-0d7b-4b0d-875d-da5c5dcb853e.svg?style=flat-square)](https://insight.sensiolabs.com/projects/7a85bc21-0d7b-4b0d-875d-da5c5dcb853e)
[![Quality Score](https://img.shields.io/scrutinizer/g/spatie/mixed-content-scanner.svg?style=flat-square)](https://scrutinizer-ci.com/g/spatie/mixed-content-scanner)
[![StyleCI](https://styleci.io/repos/28050386/shield?branch=master)](https://styleci.io/repos/28050386)
[![Total Downloads](https://img.shields.io/packagist/dt/spatie/mixed-content-scanner.svg?style=flat-square)](https://packagist.org/packages/spatie/mixed-content-scanner)

This package contains a class that can scan your site for [mixed content](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content).

Here's an example of how you can use it:

```php
use Spatie\MixedContentScanner\MixedContentScanner;

$logger = new MixedContentLogger();

$scanner = new MixedContentScanner($logger);

$scanner->scan('https://example.com');
```

`MixedContentLogger` is a class that contains methods that get called when mixed content is (not) found. 

If you don't need a custom implementation but simply want to look for mixed content using a command line tool, take a look at [our mixed-content-scanner-cli package](https://github.com/spatie/mixed-content-scanner-cli).

## Support us

Learn how to create a package like this one, by watching our premium video course:

[![Laravel Package training](https://spatie.be/github/package-training.jpg)](https://laravelpackage.training)

We invest a lot of resources into creating [best in class open source packages](https://spatie.be/open-source). You can support us by [buying one of our paid products](https://spatie.be/open-source/support-us).

We highly appreciate you sending us a postcard from your hometown, mentioning which of our package(s) you are using. You'll find our address on [our contact page](https://spatie.be/about-us). We publish all received postcards on [our virtual postcard wall](https://spatie.be/open-source/postcards).

## Installation

You can install the package via composer:

```bash
composer require spatie/mixed-content-scanner
```

## How it works under the hood

When scanning a site, the scanner will crawl everypage. On the retrieve html, these elements and attributes will be checked:

- `audio`: `src`
- `embed`: `src`
- `form`: `action`
- `link`: `href`
- `iframe`: `src`
- `img`: `src`, `srcset`
- `object`: `data`
- `param`: `value`
- `script`: `src`
- `source`: `src`, `srcset`
- `video`: `src`

If any of those attributes start with `http://` the element will be regarded as mixed content.

The package does not scan linked `.css` or `.js` files, nor does it take inline `<script>` or `<style>` and [shortlinks](http://microformats.org/wiki/rel-shortlink) into consideration.

## Usage

```php
use Spatie\MixedContentScanner\MixedContentScanner

$logger = new MixedContentLogger();

$scanner = new MixedContentScanner($logger);

$scanner->scan('https://example.com');
```

That `MixedContentScanner` accepts an instance of a class that extends `\Spatie\MixedContentScannerMixedContentObserver`. You should create such a class yourself. Let's take a look at an example implementation.

```php
use Psr\Http\Message\UriInterface;
use Spatie\MixedContentScanner\MixedContent;
use Spatie\MixedContentScanner\MixedContentObserver;

class MyMixedContentLogger extends MixedContentObserver
{
    /**
     * Will be called when mixed content was found.
     * 
     * @param \Spatie\MixedContentScanner\MixedContent $mixedContent
     */
    public function mixedContentFound(MixedContent $mixedContent)
    {
    }

    /**
     * Will be called when no mixed content was found on the given url.
     * 
     * @param \Psr\Http\Message\UriInterface $crawledUrl
     */
    public function noMixedContentFound(UriInterface $crawledUrl)
    {
    }

    /**
     * Will be called when the scanner has finished crawling.
     */
    public function finishedCrawling()
    {
    }
}
```

Of course you should supply a function body to these methods yourself. If you don't need a function just leave it off.

The `$mixedContent` variable the `mixedContentFound` class accept is an instance of `\Spatie\MixedContentScanner\MixedContent` which has these three properties:

- `$elementName`: the name of the element that is regarded as mixed content
- `$mixedContentUrl`: the url of the element that is regarded as mixed content. For an image this can be the value of `src` or `srcset` for a `form` this can be the value of `action`, ...
- `$foundOnUrl`: the url where the mixed content was found

### Customizing the requests

The scanner is powered by [our homegrown Crawler](https://github.com/spatie/crawler) which on it's turn leverages [Guzzle](http://docs.guzzlephp.org/en/stable/) to perform webrequests.
You can pass an array of options to the second argument of `MixedContentScanner`. These options will be passed to the Guzzle Client. 

Here's an example where ssl verification is being turned off.

```php
$scanner = new MixedContentScanner($logger);
$scanner->scan('https://laravel.com', ['verify' => 'false']);
```

### Filtering the crawled urls

By default the mixed content scanner will crawl all urls of the hostname given. If you want to filter the urls to be crawled, you can pass the scanner a class that extends `Spatie\Crawler\CrawlProfile`.

Here's the contents of that class:

```php
namespace Spatie\Crawler;

use Psr\Http\Message\UriInterface;

abstract class CrawlProfile
{
    /**
     * Determine if the given url should be crawled.
     *
     * @param \Psr\Http\Message\UriInterface $url
     *
     * @return bool
     */
    abstract public function shouldCrawl(UriInterface $url): bool;
}
```

And here's how you can let the scanner use your profile:

```php
use Spatie\MixedContentScanner\MixedContentScanner;

$logger = new MixedContentLogger();

$scanner = new MixedContentScanner($logger);

$scanner->setCrawlProfile(new MyCrawlProfile);
```

## Customizing the crawler

The scanner is powered by [our homegrown Crawler](https://github.com/spatie/crawler). You can call any methods on the crawler before the crawling process starts by calling `configureCrawler` on a `MixedContentScanner`.

```php
use Spatie\Crawler\Crawler;
use Spatie\MixedContentScanner\MixedContentScanner;

$scanner = (new MixedContentScanner($logger))
    ->configureCrawler(function(Crawler $crawler) {
        $crawler->setConcurrency(1) // now all urls will be crawled one by one 
    });
```

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Testing

``` bash
composer test
```

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Security

If you discover any security related issues, please email freek@spatie.be instead of using the issue tracker.

## Postcardware

You're free to use this package, but if it makes it to your production environment we highly appreciate you sending us a postcard from your hometown, mentioning which of our package(s) you are using.

Our address is: Spatie, Samberstraat 69D, 2060 Antwerp, Belgium.

We publish all received postcards [on our company website](https://spatie.be/en/opensource/postcards).

## Credits

- [Freek Van der Herten](https://github.com/freekmurze)
- [All Contributors](../../contributors)

The scanner is inspired by [mixed-content-scan](https://github.com/bramus/mixed-content-scan) by [Bram Van Damme](https://github.com/bramus). Parts of his readme and code were used.

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
