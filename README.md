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

The workflow expects the SSH user to be able to write to `EC2_APP_DIR` and to run
`sudo systemctl restart livekit-meet.service` without a password. Do not store the
LiveKit API credentials in GitHub or in the deployment bundle; create
`/etc/livekit-meet.env` on the instance with `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET`,
and `LIVEKIT_URL`, and keep that file readable only by root.

### EC2 setup

Install Node.js 22 and `rsync` on the instance. Create the deployment directory
and make it writable by the deployment user (the directory must include
`$EC2_APP_DIR/releases`). Create a systemd unit at
`/etc/systemd/system/livekit-meet.service`, replacing
`ubuntu` and the app path below if your SSH user or `EC2_APP_DIR` differs:

```ini
[Unit]
Description=LiveKit Meet
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/apps/livekit-meet/current
EnvironmentFile=/etc/livekit-meet.env
Environment=NODE_ENV=production
Environment=PORT=3000
Environment=HOSTNAME=127.0.0.1
ExecStart=/usr/bin/node /home/ubuntu/apps/livekit-meet/current/server.js
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Create `/etc/livekit-meet.env` with the required LiveKit values, then enable the
service with `sudo systemctl daemon-reload && sudo systemctl enable livekit-meet`.
Use `visudo` to allow the deployment user to restart only this service without a
password (for the default `ubuntu` user, add
`ubuntu ALL=(root) NOPASSWD: /usr/bin/systemctl restart livekit-meet.service`).
Put Nginx (or another TLS reverse proxy) in front of port 3000.
Restrict SSH ingress in the EC2 security group to trusted sources.

The workflow runs from the `main` branch. If your personal repository uses a
different default branch, update the branch name in the workflow. Keep `.env` and
all private keys out of Git; configure runtime application secrets on the instance.
