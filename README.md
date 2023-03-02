# Laravel Storybook

Development setup with the purpose of developing laravel components using Storybook.

Make sure that after you follow the setup instructions, you look at our **Thoughts** and **Todo** section to understand where we want to go from here. This is a particularly complex setup and I'm pretty certain we can simplify it using event more power from docker containers.

## One time only setup

We'll be assuming docker and docker-compose are already installed and so the first step is to run our containers in the background.

```
./storybook up
```
*Dont forget to stop them when you are not using them.*

```
./storybook down
```


### Storybook

```
./storybook composer require --dev area17/blast

# Just if we need to tweak some Storybook configuration.
./storybook php artisan vendor:publish --provider="A17\Blast\BlastServiceProvider" --tag="blast-config"
```

### Tailwind

```
./storybook yarn add tailwindcss postcss autoprefixer
./storybook npx tailwindcss init -p
```

When using Vite, the now default for Laravel 10, you'll have to build the assets so they be included on the rendering.

```
./storybook yarn run build
```

### Running Storybook

```
./storybook php artisan blast:launch
```

For easier accessibility, we create a symlink named `stories` on our root folder. This is where we place our stories.

```
ln -s ./laravel/resources/views/stories ./stories
```

### Done

We are now ready to develop stories by placing them on `./stories` folder.

Here's an example.

```
@storybook([
    'name' => 'Avatar with initials',
    'args' => [
        'title'     => 'Rui Sardinha',
        'label'     => 'RS',
        'photoUrl' => '',
    ],
    'argTypes' => [
        'photoUrl' => [
            'control'     => 'text',
            'description' => 'URL of the avatar',
        ],
    ],
])

<x-backstate::avatar.single
    class="h-10 w-10"
    :title=$title
    :label=$label
    :photoUrl=$photoUrl
/>
```


## Setting up component packages

A very good use case for this project is the development of components for redistributable packages. Let's see how we can do a setup for this.

```
mkdir laravel/packages
ln -s ./laravel/packages ./packages

# Example repository with components
git clone https://github.com/26B/backstate/ ./packages/backstate
```

Add the repository to your `./laravel/composer.json`.
```json
"repositories": [
  {
      "type": "path",
      "url": "./packages/backstate"
  },
  {
      "type": "composer",
      "url": "https://packagist.org"
  },
  {
      "packagist": false
  }
],
```

And now we can require the package.

```
./storybook composer require --dev "26b/backstate": "@dev",
```



## TODO

- Evolve by replacing some of the complexity with the simple bitnami docker-compose.yml with a custom image that extends this and include already the necessary dependencies
- The symlinks should later be replaced by docker volumes.
- Support Vite dev hot reloading instead of forcing to do a production build each time an update is necessary. (More info here: https://github.com/area17/blast)


## Thoughts

- APP_URL must be defined to localhost:8000 (same port as docker).
- ~~Updating composer currently requires a restart of the laravel container. Could be improved by using nginx instead of the internal server.~~
- Might share the `./laravel/resources/views/components` for direct component testing.
- Might share the `./laravel/tests` to add tests to the components.


## Troubleshooting

Ensure that `./laravel/.env` has the correct url and port.

```
APP_URL=http://localhost:8000
```

Ensure `./laravel/tailwind.config.js` has the content resources defined.

```json
/** @type {import('tailwindcss').Config} */
module.exports = {
    content: [
        "./resources/**/*.blade.php",
        "./resources/**/*.js",
        "./resources/**/*.vue",
    ]
}
```