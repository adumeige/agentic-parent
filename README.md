# agentic-parent

Shared Maven parent for Kotlin, Vaadin, and Embabel projects. It manages dependency
versions and build plugins, and inherits Spring Boot's parent configuration.

## Use the parent

After release `1.0.0` is available in Maven Central:

```xml
<parent>
    <groupId>io.github.adumeige.agentic-parent</groupId>
    <artifactId>agentic-parent</artifactId>
    <version>1.0.0</version>
    <relativePath/>
</parent>
```

No GitHub repository declaration or credentials are needed to resolve this parent
from Central. Child projects should declare their own Maven group and publishing
configuration. This parent's signing and Central plugins are not inherited.

## Managed project migration

The parent is prepared for these `1.0.0` coordinates. The coordinates for projects
still awaiting migration are planned destinations, not claims of availability.

| Project | Maven group | Managed artifacts | Migration status |
| --- | --- | --- | --- |
| [vaadin-themes](https://github.com/adumeige/vaadin-themes) | `io.github.adumeige.vaadin-themes` | `theme`, `theme-fjord`, `theme-novelist`, `theme-anything`, `theme-seagod`, `theme-terminal-synth`, `theme-glass`, `theme-brutalist`, `theme-analog` | Migration prepared separately |
| [vaadin-stateflow](https://github.com/adumeige/vaadin-stateflow) | `io.github.adumeige.vaadin-stateflow` | `vaadin-stateflow` | Migration prepared separately |
| [agent-tools](https://github.com/adumeige/agent-tools) | `io.github.adumeige.agent-tools` | `agent-tools` | Migration and publication needed |
| [vaadin-nvl](https://github.com/adumeige/vaadin-nvl) | `io.github.adumeige.vaadin-nvl` | `vaadin-nvl` | Migration and publication needed |
| [vaadin-pixel-charts](https://github.com/adumeige/vaadin-pixel-charts) | `io.github.adumeige.vaadin-pixel-charts` | `vaadin-pixel-charts` | Migration and publication needed |
| [vaadin-reactflow](https://github.com/adumeige/vaadin-reactflow) | `io.github.adumeige.vaadin-reactflow` | `vaadin-reactflow-component` | Migration and publication needed |
| [vaadin-graph](https://github.com/adumeige/vaadin-graph) | `io.github.adumeige.vaadin-graph` | `vaadin-graph-component`, `vaadin-graph-karibu` | Migration and publication needed |

Every managed internal artifact targets `1.0.0`. Consumers should update their
dependency group IDs when adopting this parent. Dependencies become usable when
the corresponding projects publish those versions.

Publish this parent first, then migrate and publish projects inheriting it:
`agent-tools` and `vaadin-graph` currently do so. Their managed entries do not
create a build cycle: Maven does not download unused managed dependencies merely
to resolve the parent. The other listed projects have independent parents.

`grapher.version` is reserved at `1.0.0`, but the parent currently manages no
`grapher` artifact. Grapher therefore does not block this parent migration.

The theme catalog includes `theme-analog`; a duplicate React Flow entry was removed.

## Publishing

Configure these repository Actions secrets (the same values used by the other projects):

| Secret | Value |
| --- | --- |
| `CENTRAL_USERNAME` | Sonatype Central Portal token username |
| `CENTRAL_PASSWORD` | Sonatype Central Portal token password |
| `GPG_PRIVATE_KEY` | Full ASCII-armored exported private signing key |
| `GPG_PASSPHRASE` | Signing key passphrase |

The `io.github.adumeige` namespace must be verified in Central. The public signing
key must be available on a supported keyserver, such as `keyserver.ubuntu.com`.
GitHub publication uses the built-in `GITHUB_TOKEN`; no additional secret is needed.

1. Merge the release changes into `main` and check that CI passes.
2. Open **Actions → Build and publish Agentic Parent → Run workflow**.
3. Select `main` and a new release version, initially `1.0.0`.
4. The workflow creates a versioned release commit, signs the POM, and automatically
   publishes to Central, waiting up to an hour for publication.
5. It creates an annotated `v<version>` tag and draft GitHub Release, mirrors and
   verifies the same POM and signature in GitHub Packages, attaches the artifacts
   and Central bundle, and makes the release public with generated notes.

Only a parent POM is published: there are no Java/Kotlin classes, sources JARs,
or documentation JARs in this repository. `main` retains `1.0.0-SNAPSHOT` while
the release tag points to the commit containing the actual release version.
Pushes and pull requests verify the parent and consumer inheritance; they do not
publish. Tags do not trigger another deployment. Published versions are immutable.

Central and GitHub publish sequentially. If Central succeeds and the GitHub job
fails, choose **Re-run failed jobs** on that same Actions run. The original bundle
and source are retained for 90 days. The mirror skips byte-identical files already
uploaded and refuses conflicting files. The GitHub Release remains a draft until
publication finishes, although its tag may already be visible.

Do not rerun all jobs or start a fresh run for a version already published to
Central. If Central itself fails or times out, inspect its deployment in
[Central Portal](https://central.sonatype.com/publishing/deployments) before recovery;
publication may have continued after the runner stopped. The saved bundle supports
manual recovery without rebuilding.

To verify locally without signing or publishing:

```bash
mvn -Pcentral-release verify -Dgpg.skip=true
```

Local `-Pcentral-release deploy` stages for manual Central approval by default;
the workflow explicitly enables automatic publication.
