Check the latest version of K3s using [https://github.com/k3s-io/k3s/releases](https://github.com/k3s-io/k3s/releases)

This upgrader autoupgrade your master and agent node to the latest stable version (defined by channel: https://update.k3s.io/v1-release/channels/stable)


to upgrade manually:
  master: `curl -sfL https://get.k3s.io | INSTALL_K3S_CHANNEL=latest sh -`
  node: `curl -sfL https://get.k3s.io | INSTALL_K3S_CHANNEL=latest K3S_URL="https://$IP:6443" K3S_TOKEN="$NODETOKEN" sh -`
