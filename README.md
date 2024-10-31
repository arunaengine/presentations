# Aruna presentations

[![CC BY-ND 4.0][cc-by-nd-shield]][cc-by-nd]

This repository contains additional sources, examples and explanations for presentations given in the Aruna context.

A rendered version of this repo can be found here: 

https://pres.aruna-storage.org

## Deploy Locally with Docker for Testing

* In the repository's root directory, run `docker run -v $PWD:/book -p 3000:3000 peaceiris/mdbook serve --hostname 0.0.0.0`
* Browse to http://localhost:3000
* You can edit the content while the container is running and see the results immediately.


## Contributing

The contribution to the wiki is done via pull requests. The book is rendered as markdown via the `src` directory.

[SUMMARY.md](./src/SUMMARY.md) contains the table of content for the book.

## License

This work is licensed under a
[Creative Commons Attribution-NoDerivs 4.0 International License][cc-by-nd].

[![CC BY-ND 4.0][cc-by-nd-image]][cc-by-nd]

[cc-by-nd]: https://creativecommons.org/licenses/by-nd/4.0/
[cc-by-nd-image]: https://licensebuttons.net/l/by-nd/4.0/88x31.png
[cc-by-nd-shield]: https://img.shields.io/badge/License-CC%20BY--ND%204.0-lightgrey.svg
