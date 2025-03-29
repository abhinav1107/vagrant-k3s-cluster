require 'yaml'

#  merge settings from cluster.yaml and cluster-local.yaml files
def deep_merge(original, override)
    original.merge(override) do |key, old_val, new_val|
        if old_val.is_a?(Hash) && new_val.is_a?(Hash)
            deep_merge(old_val, new_val)
        else
            new_val.nil? ? old_val : new_val
        end
    end
end

def load_cluster_config
    global_config = {}
    local_config = {}

  if File.exist?('cluster.yaml')
    begin
      global_config = YAML.load_file('cluster.yaml') || {}
    rescue Psych::SyntaxError
      warn "Failed loading cluster.yaml due to syntax error"
    end
  else
    warn "Missing cluster.yaml file"
  end

  if File.exist?('cluster-local.yaml')
    begin
      local_config = YAML.load_file('cluster-local.yaml') || {}
    rescue Psych::SyntaxError
      warn "Failed loading cluster-local.yaml due to syntax error"
    end
  end

  # Perform deep merge
  deep_merge(global_config, local_config)
end
cluster_config = load_cluster_config
cluster_name = cluster_config.key?("name") ? cluster_config["name"].downcase.gsub(/\s+/, "-") : "vagrant-k8s"

domain_name = cluster_config.key?("domain") ? cluster_config["domain"] : "localdomain"
box_image = cluster_config.key?("image") ? cluster_config["image"] : "rockylinux/9"

# controller node settings
controller_ha = cluster_config.dig("nodes", "controller", "ha") || false
controller_count = controller_ha ? 3 : 1
controller_cpu = cluster_config.dig("nodes", "controller", "cpu") || 2
controller_memory = cluster_config.dig("nodes", "controller", "memory") || 2048

# worker node settings
worker_count = cluster_config.dig("nodes", "worker", "count") || 2
worker_cpu = cluster_config.dig("nodes", "worker", "cpu") || 2
worker_memory = cluster_config.dig("nodes", "worker", "memory") || 2048

# load balancer node settings
load_balancer_count = cluster_config.dig("nodes", "loadbalancer", "count") || 0
load_balancer_cpu = cluster_config.dig("nodes", "loadbalancer", "cpu") || 1
load_balancer_memory = cluster_config.dig("nodes", "loadbalancer", "memory") || 1024
lb_enabled = cluster_config.dig("nodes", "loadbalancer", "enabled") || false

# vagrant network settings
hostonly_enabled = cluster_config.dig("networking", "hostonly", "enabled") || false
hostonly_start_ip = cluster_config.dig("networking", "hostonly", "start_ip") || ""
public_enabled = cluster_config.dig("networking", "public", "enabled") || false
public_start_ip = cluster_config.dig("networking", "public", "start_ip") || false
public_bridge = cluster_config.dig("networking", "public", "bridge") || false

node_index = 0
project_path = File.expand_path(File.dirname(__FILE__))

# Vagrant box configuration
Vagrant.configure("2") do |config|
    config.vm.box = box_image
    config.vm.box_check_update = false

    # controller creation block
    controller_count.times do |i|
        node_index += 1
        node_name = "controller-#{i + 1}"
        hostname = "#{node_name}.#{domain_name}"
        hostonly_ip = hostonly_enabled && hostonly_start_ip ? hostonly_start_ip.split('.').tap { |ip| ip[-1] = ip[-1].to_i + node_index - 1 }.join('.') : nil
        public_ip = public_enabled && public_start_ip ? public_start_ip.split('.').tap { |ip| ip[-1] = ip[-1].to_i + node_index - 1 }.join('.') : nil

        config.vm.define node_name do |vb|
            # vb.vm.hostname = hostname

            # configure host only network if given
            if hostonly_enabled
                vb.vm.network "private_network", ip: hostonly_ip if hostonly_ip
            end

            # configure public network if given
            if public_enabled
                public_network_opts = {}
                public_network_opts[:ip] = public_ip if public_ip
                public_network_opts[:bridge] = public_bridge if public_bridge
                vb.vm.network "public_network", **public_network_opts unless public_network_opts.empty?
            end

            # vm settings
            vb.vm.provider "virtualbox" do |ps|
                ps.memory = controller_memory
                ps.cpus = controller_cpu
                ps.name = "vb-k8s-#{cluster_name}-#{node_name}"
            end
        end
    end

    # worker creation block
    worker_count.times do |i|
        node_index += 1
        node_name = "worker-#{i + 1}"
        hostname = "#{node_name}.#{domain_name}"
        hostonly_ip = hostonly_enabled && hostonly_start_ip ? hostonly_start_ip.split('.').tap { |ip| ip[-1] = ip[-1].to_i + node_index - 1 }.join('.') : nil
        public_ip = public_enabled && public_start_ip ? public_start_ip.split('.').tap { |ip| ip[-1] = ip[-1].to_i + node_index - 1 }.join('.') : nil

        config.vm.define node_name do |vb|
            # vb.vm.hostname = hostname

            # configure host only network if given
            if hostonly_enabled
                vb.vm.network "private_network", ip: hostonly_ip if hostonly_ip
            end

            # configure public network if given
            if public_enabled
                public_network_opts = {}
                public_network_opts[:ip] = public_ip if public_ip
                public_network_opts[:bridge] = public_bridge if public_bridge
                vb.vm.network "public_network", **public_network_opts unless public_network_opts.empty?
            end

            # vm settings
            vb.vm.provider "virtualbox" do |ps|
                ps.memory = worker_memory
                ps.cpus = worker_cpu
                ps.name = "vb-k8s-#{cluster_name}-#{node_name}"
            end
        end
    end

    if lb_enabled
        load_balancer_count.times do |i|
            node_index += 1
            node_name = "lb-#{i + 1}"
            hostname = "#{node_name}.#{domain_name}"
            hostonly_ip = hostonly_enabled && hostonly_start_ip ? hostonly_start_ip.split('.').tap { |ip| ip[-1] = ip[-1].to_i + node_index - 1 }.join('.') : nil
            public_ip = public_enabled && public_start_ip ? public_start_ip.split('.').tap { |ip| ip[-1] = ip[-1].to_i + node_index - 1 }.join('.') : nil

            config.vm.define node_name do |vb|
                # vb.vm.hostname = hostname

                # configure host only network if given
                if hostonly_enabled
                    vb.vm.network "private_network", ip: hostonly_ip if hostonly_ip
                end

                # configure public network if given
                if public_enabled
                    public_network_opts = {}
                    public_network_opts[:ip] = public_ip if public_ip
                    public_network_opts[:bridge] = public_bridge if public_bridge
                    vb.vm.network "public_network", **public_network_opts unless public_network_opts.empty?
                end

                # vm settings
                vb.vm.provider "virtualbox" do |ps|
                    ps.memory = load_balancer_memory
                    ps.cpus = load_balancer_cpu
                    ps.name = "vb-k8s-#{cluster_name}-#{node_name}"
                end
            end
        end
    end
end
