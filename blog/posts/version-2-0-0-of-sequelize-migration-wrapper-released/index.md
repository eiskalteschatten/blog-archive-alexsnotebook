After almost five years of no updates, I finally released version 2.0.0 of [sequelize-migration-wrapper](https://github.com/eiskalteschatten/sequelize-migration-wrapper), my npm package for Sequelize migrations. The package is more or less just a wrapper with common functionality for the [Sequelize](https://www.npmjs.com/package/sequelize) migration tool, [Umzug](https://www.npmjs.com/package/umzug).

These are the release notes:

-   Major update for `umzug`
-   **Breaking Change:** a glob is now required instead of the `path`/`filePattern` combination
-   Security updates

Do note though that there is a breaking change between this version and the previous version. That is highlighted in the list above.

You can find it here:

-   GitHub: [https://github.com/eiskalteschatten/sequelize-migration-wrapper](https://github.com/eiskalteschatten/sequelize-migration-wrapper)
-   npm: [https://www.npmjs.com/package/sequelize-migration-wrapper](https://www.npmjs.com/package/sequelize-migration-wrapper?activeTab=readme)