<img width="1453" height="937" alt="image" src="https://github.com/user-attachments/assets/80424c40-13a4-4756-a08a-26b81c90b0de" />
<img width="1459" height="929" alt="image" src="https://github.com/user-attachments/assets/09c906c6-f3ce-4a79-85bc-cdee5a2ca7f3" />


# sv

Everything you need to build a Svelte project, powered by [`sv`](https://github.com/sveltejs/cli).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```sh
# create a new project in the current directory
npx sv create

# create a new project in my-app
npx sv create my-app
```

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```sh
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```sh
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://svelte.dev/docs/kit/adapters) for your target environment.
