# vagrant-convoy
vagrant file for convoy

$vagrant up
$vagrant ssh node0
$sudo docker run -v vol1:/vol1 --volume-driver=convoy ubuntu touch /vol1/foo

- ref: [convoy quick start guid](https://github.com/rancher/convoy#quick-start-guide)
