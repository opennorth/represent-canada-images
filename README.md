# Represent API: Images

[Represent](https://represent.opennorth.ca/) is the open database of Canadian elected officials and electoral districts. It provides a [REST API](https://represent.opennorth.ca/api/) to boundary, representative, and postcode resources.

This repository stores scripts to extract images from the database. The [represent-canada](https://github.com/opennorth/represent-canada) repository is what's running at [represent.opennorth.ca](https://represent.opennorth.ca/).

## Usage

Download and resize images:

    bundle
    rake download
    rake resize

You now have the original images in `images/original` and 60x90 images in `60x90`. `60x90` was chosen because the smallest image is 63 pixels wide and 95 pixels high and the most common aspect ratio is 2:3.

Consult the image sizes and aspect ratios:

    rake sizes
    rake ratios

Create an HTML page with all the 60x90 images:

    rake html
    open images.html

Remove all images:

    rake clean

## Contributing

If you would like to retrieve more photos with these scripts, improve our [Canadian legislative scrapers](https://github.com/opencivicdata/scrapers-ca/) to scrape images from legislative websites.

## Bugs? Questions?

This project's main repository is on GitHub: [https://github.com/opennorth/represent-canada-images](https://github.com/opennorth/represent-canada-images), where your contributions, forks, bug reports, feature requests, and feedback are greatly welcomed.

Copyright (c) 2014 Open North Inc., released under the MIT license
