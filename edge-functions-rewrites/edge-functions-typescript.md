QUESTION
Hi Team, Launch Edge Functions are JavaScript-only per the published docs. Our codebase is TypeScript throughout. Is TypeScript support planned? What is the current recommended workaround?

ANSWER
Launch Edge Functions run JavaScript only. They do not run TypeScript directly in the Edge environment.

What you can do: Write your Edge Functions in TypeScript in your project, then turn them into JavaScript during your build. Put the built JavaScript files in the functions folder, as Launch expects. This is the same idea many teams use elsewhere—TypeScript for authoring, JavaScript for what actually runs.

Example tsconfig.json you can start from:

{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "bundler",
    "outDir": "./functions",
    "rootDir": "./src/edge",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/edge/**/*"]
}
