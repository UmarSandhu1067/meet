<a href="https://livekit.io/">
  <img src="./.github/assets/livekit-mark.png" alt="LiveKit logo" width="100" height="100">
</a>

# LiveKit Meet

<p>
  <a href="https://meet.livekit.io"><strong>Try the demo</strong></a>
  •
  <a href="https://github.com/livekit/components-js">LiveKit Components</a>
  •
  <a href="https://docs.livekit.io/">LiveKit Docs</a>
  •
  <a href="https://livekit.io/cloud">LiveKit Cloud</a>
  •
  <a href="https://blog.livekit.io/">Blog</a>
</p>

<br>

LiveKit Meet is an open source video conferencing app built on [LiveKit Components](https://github.com/livekit/components-js), [LiveKit Cloud](https://cloud.livekit.io/), and Next.js. It's been completely redesigned from the ground up using our new components library.

![LiveKit Meet screenshot](./.github/assets/livekit-meet.jpg)

## Tech Stack

- This is a [Next.js](https://nextjs.org/) project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app).
- App is built with [@livekit/components-react](https://github.com/livekit/components-js/) library.

## Demo

Give it a try at https://meet.livekit.io.

## Dev Setup

Steps to get a local dev setup up and running:

1. Run `pnpm install` to install all dependencies.
2. Copy `.env.example` in the project root and rename it to `.env.local`.
3. Update the missing environment variables in the newly created `.env.local` file.
4. Run `pnpm dev` to start the development server and visit [http://localhost:3000](http://localhost:3000) to see the result.
5. Start development 🎉

## Deploy to AWS EC2

The GitHub Actions workflow in `.github/workflows/deploy-ec2.yaml` checks out Git
LFS assets, builds the Next.js standalone app, and deploys it whenever `main` is
updated (or when run manually).
It connects to an Ubuntu EC2 instance over SSH and switches the `current` release
only after uploading the build.

### GitHub configuration

In your personal repository, open **Settings → Secrets and variables → Actions** and
add these repository variables:

- `EC2_HOST`: the instance's public DNS name or IP address.
- `EC2_USER`: the SSH user (usually `ubuntu`).
- `EC2_APP_DIR`: an absolute deployment directory, such as
  `/home/ubuntu/apps/livekit-meet` (do not include spaces).

Add these repository secrets:

- `EC2_SSH_KEY`: the private SSH key for the deployment user.
- `EC2_KNOWN_HOSTS`: the verified SSH host-key line(s) for the EC2 instance.

The workflow expects `EC2_USER` to be the same Ubuntu user that owns the PM2
process named `livekit-meet`, and to be able to write to `EC2_APP_DIR`. The PM2
process must run the standalone server through
`$EC2_APP_DIR/current/server.js`, so switching releases updates the app it starts.
Do not store LiveKit API credentials in GitHub or in the deployment bundle; set
`LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET`, and `LIVEKIT_URL` in the PM2 process
environment on the instance.

### EC2 setup

Install Node.js 22, `rsync`, and PM2 on the instance. Create `EC2_APP_DIR` and
make it writable by the PM2 user. Configure the existing PM2 process named
`livekit-meet` to run
`$EC2_APP_DIR/current/server.js` with `$EC2_APP_DIR/current` as its working
directory, `PORT=3000`, and `HOSTNAME=127.0.0.1`. Keep the LiveKit credentials in
that process's environment, not in the repository.

Run `pm2 startup` once on the instance and execute the exact `sudo` command it
prints, then run `pm2 save` so the process is restored after an EC2 reboot. Put
Nginx (or another TLS reverse proxy) in front of port 3000.
Restrict SSH ingress in the EC2 security group to trusted sources.

The workflow runs from the `main` branch. If your personal repository uses a
different default branch, update the branch name in the workflow. Keep `.env` and
all private keys out of Git; configure runtime application secrets on the instance.
