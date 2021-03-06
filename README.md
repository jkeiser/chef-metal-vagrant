# chef-provisioning-vagrant

[![Gem Version](https://badge.fury.io/rb/chef-provisioning-vagrant.svg)](https://badge.fury.io/rb/chef-provisioning-vagrant) [![Build Status](https://travis-ci.org/chef/chef-provisioning-vagrant.svg?branch=master)](https://travis-ci.org/chef/chef-provisioning-vagrant)

This is the Vagrant driver for chef-provisioning.

# Resources

## vagrant_box

Specifying this resource will verify that the correct vagrant box is downloaded to the box directory (default of `~/.chefdk/vms` when using the chefdk) for the correct vagrant provider. The vagrant provider defaults to virtualbox.

This example will `vagrant box add` the box if it is not currently on your system.

```ruby
vagrant_box 'opscode-centos-6.4' do
  url 'http://opscode-vm-bento.s3.amazonaws.com/vagrant/vmware/opscode_centos-6.4_chef-provisionerless.box'
  vagrant_provider 'vmware_desktop'
end
```

This example will ensure that the vmware_desktop/fusion based box exists on your system, and will fail if the box does not exist. If this fails, you can always add the URL source as per the previous example. **Note: since bento boxes appear as 'vmware_desktop', 'vmware_fusion' will not work here**

```ruby
vagrant_box 'custom_box' do
  vagrant_provider 'vmware_desktop'
end
```

This example will ensure that the virtualbox based box already exists on your system, and will fail if the box does not exist. As before, adding the URL will cause it to download the box.

```ruby
vagrant_box 'custom_box'
```

# Machine Options

An example of machine options would be as follows:

```ruby
options = {
  vagrant_options: {
    'vm.box' => 'opscode-centos-6.4',
    # becomes
    # config.vm.network(:private_network, {ip: "22.22.22.22"})
    # config.vm.network(:forwarded_port, guest: 4001, host: 4001)
    # config.vm.network(:forwarded_port, guest: 3001, host: 3001)
    # in Vagrantfile:
    'vm.network' => [
      ':private_network, {ip: "22.22.22.22"}',
      ':forwarded_port, guest: 4001, host: 4001',
      ':forwarded_port, guest: 3001, host: 3001'
    ]
  },
  # `vagrant_config` gets appended to the Vagrantfile
  vagrant_config: <<EOF
    config.vm.provider 'virtualbox' do |v|
      v.memory = 4096
      v.cpus = 2
    end
  EOF
}

machine 'marley' do
  machine_options options
  converge true
end
```

You can also add a `vagrant_provider` attribute to override the default virtualbox provider:

```ruby
options = {
  vagrant_provider: 'vmware_fusion'
  vagrant_options: {
    'vm.box' => 'opscode-centos-6.4',
  },
}

machine 'marley' do
  machine_options options
  converge true
end
```

**Note: even though the bento based boxes appear as 'vmware_desktop', 'vmware_fusion' is required here, as it is the name of the vagrant provider**

You can also add a `vagrant_config` attribute to specify provider specific options. For example, to specify a memory setting of 1536 MB in a virtual box provider:

```ruby
options = {
  vagrant_config: "config.vm.provider \"virtualbox\" do |vb|\n  vb.memory = 1536\nend\n",
  vagrant_options: {
    'vm.box' => 'opscode-centos-6.4'
  }
}

machine 'marley' do
  machine_options options
  converge true
end
```

# Known Issues

It would be really nice to do some magic to make the vmware_fusion vs vmware_desktop providers match in the machine_options and the vagrant_box resource, but some magic would happen there...
