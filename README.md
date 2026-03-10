### Running locally
1. Install ruby (Windows: `winget install RubyInstallerTeam.RubyWithDevKit.3.2`)
2. Install bundler (`gem install bundler`)
3. Install dependencies (`bundle install`)
4. Run (`bundle exec jekyll serve`)
5. From a device on the same network, visit `http://<your-PC-IP>:4000/`. From the device hosting it, use `127.0.0.1` or `localhost` as the IP.
6. If it doesn't load, allow inbound access to port `4000` in firewall
