---
title: SonarQube
weight: 30
---

The widget displays SonarQube metrics.

## Authorization

Authorization is configured in [External services](../../external-services/#sonarqube).

## Configuration

| Name        | Required | Description                                                               | Default                               |
| ----------- | -------- | ------------------------------------------------------------------------- | ------------------------------------- |
| URL         | Yes      | SonarQube URL, for example, `https://sonarqube.example.com`               | -                                     |
| Project key | Yes      | Project identifier in SonarQube                                           | -                                     |
| Branch      | No       | Project branch from which metrics are retrieved                           | According to the SonarQube project settings |
| Metrics     | Yes      | Project metrics displayed in the widget. Specify each metric key in the configuration |                               |

See the [list of available metrics](https://docs.sonarsource.com/sonarqube-server/latest/user-guide/code-metrics/metrics-definition) for the current SonarQube version.

## Additional widget features

The widget can display data for the default branch or any other branch.
