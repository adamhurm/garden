---
created: 2025-05-15T22:40
updated: 2025-07-21T18:39
tags:
  - homeassistant
  - homelab
---
[MediaTracker](https://github.com/bonukai/MediaTracker) is a "self hosted platform \[created by [bonukai](https://github.com/bonukai)\] for tracking movies, tv shows, video games, books and audiobooks".

---
## Deploying in kubernetes
I created a kubernetes deployment for my homelab cluster in [this commit](https://github.com/adamhurm/homelab/commit/03a9bc082a512ff26f5a49ae672287f6b155f41b) (this repo is currently private but I plan to open it up soon).

## HomeAssistant installation 
Install [HACS](https://hacs.xyz/docs/use/) add-on in order to set up HACS:
https://my.home-assistant.io/redirect/supervisor_addon/?addon=cb646a50_get&repository_url=https%3A%2F%2Fgithub.com%2Fhacs%2Faddons
- Go to HomeAssistant Add-ons and start HACS: `https://ha.my.domain:8123/hassio/dashboard`
- Restart HomeAssistant. 
- Add [HACS as an Integration](https://hacs.xyz/docs/use/configuration/basic/#prerequisites).
- Under HACS dashboard: `https://ha.my.domain:8123/hacs/dashboard`
	- ... -> Custom Repositories -> Add custom repo URLs:
		- https://github.com/jonkristian/mediatracker-ha
		- https://github.com/jonkristian/mediatracker-ha-car
- Add [Integration](https://ha.my.domain:8123/config/integrations/dashboard) and select MediaTracker.
	- Create app token to use for this step: `https://mediatracker.my.domain/#/settings/application-tokens`
- Add the MediaTracker Card to a Dashboard in HomeAssistant:
```yaml
views:
  - path: default_view
    title: Daily View
    cards:
      - name: Upcoming Shows / Movies
        type: custom:mediatracker-card
        entities:
          - entity: calendar.tv_series
          - entity: calendar.movies
        style: backdrop
        number_of_days: '7'
        refresh_interval: '60'
        description: week
        source_links: primary
        human_readable_countdown: true
        external_media: true
        constrict_height: false
        show_rating: false
```

