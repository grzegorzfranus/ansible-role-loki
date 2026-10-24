# Changelog

## [0.4.1](https://github.com/grzegorzfranus/ansible-role-loki/compare/v0.4.0...v0.4.1) (2026-10-24)


### CI/CD

* **workflows:** align Dependabot, release and CI workflows with github-workflows ([#14](https://github.com/grzegorzfranus/ansible-role-loki/issues/14)) ([#15](https://github.com/grzegorzfranus/ansible-role-loki/issues/15)) ([f78229b](https://github.com/grzegorzfranus/ansible-role-loki/commit/f78229b8971357634fae7902e5990c39b9a4a396))

## [0.4.0](https://github.com/grzegorzfranus/ansible-role-loki/compare/v0.3.0...v0.4.0) (2026-10-24)


### ⚠ BREAKING CHANGES

* **platforms:** drop Debian 11 (Bullseye) support ([#11](https://github.com/grzegorzfranus/ansible-role-loki/issues/11)) (#12)

### Features

* **platforms:** drop Debian 11 (Bullseye) support ([#11](https://github.com/grzegorzfranus/ansible-role-loki/issues/11)) ([#12](https://github.com/grzegorzfranus/ansible-role-loki/issues/12)) ([1265183](https://github.com/grzegorzfranus/ansible-role-loki/commit/126518339199cfb2f75b24de86bcca273406ad0a))

## [0.3.0](https://github.com/grzegorzfranus/ansible-role-loki/compare/v0.2.1...v0.3.0) (2026-08-18)


### Features

* **logrotate:** add archive directory, date suffix and time-based retention ([#8](https://github.com/grzegorzfranus/ansible-role-loki/issues/8)) ([a954496](https://github.com/grzegorzfranus/ansible-role-loki/commit/a9544964cb81f8a55dbcda1483341d89245929c0))

## [0.2.1](https://github.com/grzegorzfranus/ansible-role-loki/compare/v0.2.0...v0.2.1) (2026-08-04)


### Bug Fixes

* **docker:** add become to docker_image_pull task ([#5](https://github.com/grzegorzfranus/ansible-role-loki/issues/5)) ([3526017](https://github.com/grzegorzfranus/ansible-role-loki/commit/352601788a9b8c643219d427d1b53d97ba345acc))

## [0.2.0](https://github.com/grzegorzfranus/ansible-role-loki/compare/v0.1.0...v0.2.0) (2026-07-31)


### Features

* initial Loki role with Docker Compose and binary deployment modes ([#2](https://github.com/grzegorzfranus/ansible-role-loki/issues/2)) ([06fbb1a](https://github.com/grzegorzfranus/ansible-role-loki/commit/06fbb1a2f2e3bc1706f76dc4d3412b13ec35ead7))
