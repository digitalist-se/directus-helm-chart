# Digitalist Directus Helm Chart

This is the Digitalist Helm chart for [Directus](https://directus.io/).
Based on the [Directus Community Helm Charts](https://github.com/directus-community/helm-chart).

## Documentation

Chart documentation is found in the [chart directory](charts/directus/README.md).




Snapshots idé:

Snapshot pod
DIFF=$(diff latest-state.yaml state.yaml)
if [ "$DIFF" != "" ]; then echo "modified"; fi

det vill säga, startup, skapa state.yaml. sedan 1 gång per var 5 minut efter sleep, gör ny export.
Om diff, ladda upp nya filen till storage, byt ut startup.yaml mot nya exporten.

Vidareutveckling:
* Lägg snapshot.yaml som configmap
* Importera med pre-upgrade hook om definerad.
  https://helm.sh/docs/topics/charts_hooks/
