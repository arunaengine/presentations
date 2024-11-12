<p align=center>
<img width=300 src="./assets/aruna_white_font.png" alt="Aruna logo">
</p>

[![CC BY-ND 4.0][cc-by-nd-shield]][cc-by-nd]

Welcome to our presentation repo, where you'll find extra info on the topics we've presented. This is meant to give you a more in-depth look at the material, with additional examples, contexts, and more.

If you are looking for technical documentation for aruna itself please click [here](https://docs.aruna-engine.org/latest/).


> [!NOTE]
> [Click to see our latest KubeCon + CloudNativeCon information](./src/k8s_mortality.md)

A rendered version of this repo can be found here: 

[https://info.aruna-engine.org](https://info.aruna-engine.org)

## Developer info

### Deploy Locally with Docker for Testing

* In the repository's root directory, run `docker run -v $PWD:/book -p 3000:3000 peaceiris/mdbook serve --hostname 0.0.0.0`
* Browse to http://localhost:3000
* You can edit the content while the container is running and see the results immediately.


### Contributing

The contribution to the wiki is done via pull requests. The book is rendered as markdown via the `src` directory.

[SUMMARY.md](./SUMMARY.md) contains the table of content for the book.

### License

This work is licensed under a
[Creative Commons Attribution-NoDerivs 4.0 International License][cc-by-nd].

[![CC BY-ND 4.0][cc-by-nd-image]][cc-by-nd]

[cc-by-nd]: https://creativecommons.org/licenses/by-nd/4.0/
[cc-by-nd-image]: https://licensebuttons.net/l/by-nd/4.0/88x31.png
[cc-by-nd-shield]: https://img.shields.io/badge/License-CC%20BY--ND%204.0-lightgrey.svg
