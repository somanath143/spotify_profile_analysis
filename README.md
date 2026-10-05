# Spotify Profile

> A web app for visualizing personalized Spotify data

**Built by [somanath goudar](https://github.com/somanath143)**

## Tech Stack

- [Spotify Web API](https://developer.spotify.com/documentation/web-api/)
- [Create React App](https://github.com/facebook/create-react-app)
- [Express](https://expressjs.com/)
- [Reach Router](https://reach.tech/router)
- [Styled Components](https://www.styled-components.com/)

## Setup

1. [Register a Spotify App](https://developer.spotify.com/dashboard/applications) and add `http://localhost:8888/callback` as a Redirect URI in the app settings
1. Create an `.env` file in the root of the project based on `.env.example`
1. `nvm use`
1. `yarn && yarn client:install`
1. `yarn dev`

## Deploying to Render

1. Push your code to GitHub

2. Go to [Render](https://render.com) and create a new **Web Service** connected to your repo

3. Set the following:
   - **Build Command:** `yarn && cd client && yarn && yarn build`
   - **Start Command:** `yarn start`

4. Add environment variables:

   ```
   CLIENT_ID=your_spotify_client_id
   CLIENT_SECRET=your_spotify_client_secret
   REDIRECT_URI=https://your-app-name.onrender.com/callback
   FRONTEND_URI=https://your-app-name.onrender.com
   ```

5. Add `https://your-app-name.onrender.com/callback` as a Redirect URI in the Spotify application settings

6. Once the app is live on Render, navigate to `https://your-app-name.onrender.com/login` to start using it
