# ELUNA 2019 DevDay+: Advanced Development Workshop

## Day 0 Preparation:

### Initial setup
1. Fork this repostory to your Github account
1. Download and Install [Docker Community Edition](https://www.docker.com/products/docker-engine) for your machine platform.
1. Clone this repository: `git clone https://github.com/[myuser]/eluna-2019-devday-advanced-workshop` (if you would like to make your own modifications, you can do so by forking the repository).

### Pull Docker images

To speed up the setup of your docker image, you can pull prebuilt versions the images to your computer. This is highly recommended since docker images can take a long time to build, and we cannot guarrantee that these large images will be fast to download in the convention center.

```sh
docker pull quay.io/nyulibraries/eluna-2019-devday-advanced-workshop:pt-1
docker pull quay.io/nyulibraries/eluna-2019-devday-advanced-workshop:pt-2
docker pull quay.io/nyulibraries/eluna-2019-devday-advanced-workshop:pt-3
docker pull quay.io/nyulibraries/eluna-2019-devday-advanced-workshop:e2e
```

### Getting started in Docker
1. With the Docker daemon running, build the Docker image.
      ```sh
      docker-compose build
      ```
1. Run the container service
      ```sh
      # The 'up' command will deploy any dependent services,
      # and run with the service's ports enabled and mapped to the host.
      docker-compose up web
      ```
1. Navigate to `http://localhost:8003/primo-explore/search?vid=NYU` to see the basic VIEW up and running!

### Making changes using volumes

Notice that in `docker-compose.yml` there is a `volumes` section defined for the `web` service. Here, your local directory is being mounted as a 'volume' to the container at the specified directory:

```yml
services:
  web:
    #...
    volumes:
    - ./primo-explore/custom:/app/primo-explore/custom/
```

The gulp process will therefore dynamically respond to changes in the files in the local filesystem, which are reflected in the container and re-run there. To test that this is working, change some files and watch the page refresh and reflect those changes. For example:

`NYU/main.js`
```js
app.run(() => {
  console.log('hello from the main.js!');
});
```
`NYU/scss/main.scss`
```scss
p {
  color: red;
}
```

### Adding an `npm` dependency

One common way of delivering and using other institutions' customizations is through the [npm repository](http://npmjs.com/). Let's try adding one of these customizations.

1. Ensure your `custom` folder is mounted as a volume, then open an Alpine shell process in the Docker container defined by the `web` service
      ```sh
      docker-compose run web sh # Opens a shell within the container
      ```
1. Navigate to the `NYU` directory and add the dependency with yarn.
      ```sh
      # Within the container...
      cd primo-explore/custom/NYU
      yarn add primo-explore-search-bar-sub-menu
      ```

      You will then see the changes reflected in your `package.json` file (read with `cat package.json`)!

      ```js
      {
        "name": "primo-explore-nyu",
        "version": "1.0.0",
        "description": "This the NYU Consortium primo-explore view package.",
        "main": "js/main.js",
        "dependencies": {
          "primo-explore-search-bar-sub-menu": "^1.0.7"
        }
      }
      ```
1. To implement this in your running development instance, we can [follow the module's documentation](primo-explore-search-bar-sub-menu)

      ```js
      // Imports the module
      import 'primo-explore-search-bar-sub-menu';

      // Defines the modules on the angular instance
      let app = angular.module('viewCustom', [
        'searchBarSubMenu',
      ]);

      // Configures the modules
      app.constant('searchBarSubMenuItems', [{
          name: "Provide Feedback",
          description: "Provide Feedback",
          action: "https://nyu.qualtrics.com/jfe/form/SV_blQ3OFOew9vl6Pb?Source=NYU",
          icon: {
            set: 'communication',
            icon: 'ic_forum_24px'
          }
        },
        {
          name: "Library Hours",
          description: "Library Hours",
          action: "https://guides.nyu.edu/library-hours",
          icon: {
            set: 'av',
            icon: 'ic_av_timer_24px'
          }
        }
      ]);

      // 'Injects' the module
      app.component('prmSearchBarAfter', {
        template: '<search-bar-sub-menu></search-bar-sub-menu>'
      });
      ```
### Create a package

Now, creating a package is as simple using a single command. And Docker containers will help guarrantee that your output is not going to be effected by variations in environment!

```sh
VIEW=[VIEW] docker-compose run create-package
```

For now, the `VIEW` value can be either `NYU` or `CENTRAL_PACKAGE`.

### CircleCi account

Create an account at [circleci.com](https://circleci.com) using your Github account.

### Other local requirements

Many applications that you would need to do can now be run in the Docker container instead of on your local machine. However, there may be some processes that it would be more convenient to run locally. For example, you may want to run the development environment locally instead. To do this, I recommend installing `Node` to an LTS version. For better forward-compatibility, I have enforced a version requirement `>=8`. We'll also need these dependencies locally installed in order to properly run the Cypress GUI.

Recommended installation:
* Node (>= version 8)
* [Node Version Manager](https://github.com/creationix/nvm) for handling multiple versions of Node on your local machine (or, [nvm-windows](https://github.com/coreybutler/nvm-windows))
* [yarn](https://yarnpkg.com/lang/en/docs/install/) for node dependency management

    Once this is complete, you can simply run your `yarn` and `lerna` commands locally instead, since `yarn.lock` and `package.json` file generation is generally not platform-specific.

    ```sh
    # Now can be done locally!
    cd primo-explore/custom/NYU
    yarn add primo-explore-search-bar-sub-menu
    ```

* Google Chrome (latest version): Our section on end-to-end testing will use [cypress](https://www.cypress.io/), which utilizes Google Chrome for its testing GUI.
* Try to make sure the Cypress GUI works: `yarn global add cypress`, `cypress verify`, and `cypress open`.
