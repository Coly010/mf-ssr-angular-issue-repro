## Issue with SSR Module Federation and Angular

This is a reproduction of the issue with SSR Module Federation and Angular.

### Steps to reproduce

1. Run `npx nx serve ngshell`
2. Open `http://localhost:4200/remote2`
3. Note in the Network tab that remote2 HTML does not contain server side rendered content

### Expected behavior

The remote2 HTML should contain server side rendered content.

### Places to check

The Module Federation Plugin is configured in the `withModuleFederationForSSR` function in `webpack.server.config.ts`.
This function exists in `node_modules/@nx/angular/src/utils/mf/with-module-federation-ssr.ts`.

### What I've attempted

I've tried to set library.type to `commonjs-module` in the `withModuleFederationForSSR` function.

Such that the `withModuleFederationForSSR` function looks like this:

```ts
const updatedConfig = {
   ...(config ?? {}),
   target: 'async-node',
   output: {
       ...(config.output ?? {}),
       uniqueName: options.name,
     library: {
         ...(config.output.library ?? {}),
       type: 'commonjs-module'
     }
   },
   optimization: {
       ...(config.optimization ?? {}),
       runtimeChunk: false,
   },
   resolve: {
       ...(config.resolve ?? {}),
       alias: {
           ...(config.resolve?.alias ?? {}),
           ...sharedLibraries.getAliases(),
       },
   },
   plugins: [
       ...(config.plugins ?? []),
       new (require('@module-federation/enhanced').ModuleFederationPlugin)({
           name: options.name.replace(/-/g, '_'),
           filename: 'remoteEntry.js',
           exposes: options.exposes,
           remotes: mappedRemotes,
           shared: {
               ...sharedDependencies,
           },
         library: {
             type: 'commonjs-module'
         },
         remoteType: 'script',
         isServer: true,
           /**
            * Apply user-defined config override
            */
           ...(configOverride ? configOverride : {}),
           runtimePlugins: process.env.NX_MF_DEV_REMOTES &&
               !options.disableNxRuntimeLibraryControlPlugin
               ? [
                   ...(configOverride?.runtimePlugins ?? []),
                   require.resolve('@module-federation/node/runtimePlugin'),
                   require.resolve('@nx/webpack/src/utils/module-federation/plugins/runtime-library-control.plugin.js'),
               ]
               : [
                   ...(configOverride?.runtimePlugins ?? []),
                   require.resolve('@module-federation/node/runtimePlugin'),
               ],
           virtualRuntimeEntry: true,
       }, {}),
       sharedLibraries.getReplacementPlugin(),
   ],
};
```
