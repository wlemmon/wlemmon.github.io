<!-- Copy and paste the converted output. -->

<h1>Ray Cheatsheet</h1>
<p>
Starting or re-wetting feet in a framework can be a struggle, which is why I
present here a cheatsheet, tips, and tricks for launching ray clusters in AWS.
</p>
<h2>Initial setup</h2>
<h3>Devbox setup</h3>
<p>
I use an ec2 instance as my dev-box.
</p>
<ul>
<li>when you ssh into the box, reverse forward port 5900 (for x11vnc)
<li>pip install ray[rllib]==0.8.5
<li>sudo apt-get install chromium-browser
<li>run aws configure.
</li>
</ul>
<p>
<em>~/.bashrc</em>
</p>


<pre
class="prettyprint"># looks up head public ip, converts it to private ip
function headip(){
local RAY_HEAD_IP=$(ray get-head-ip tune-default.yaml);
RAY_HEAD_IP=$(aws ec2 describe-instances --filters "Name=instance-lifecycle,Values=spot,Name=ip-address,Values=$RAY_HEAD_IP" | jq -r '.Reservations[0].Instances[0].NetworkInterfaces[0].PrivateIpAddress')
echo $RAY_HEAD_IP
}</pre>
<p>
<em>config.yaml</em>
</p>


<pre
class="prettyprint">cluster_name: tune-default
provider: {type: aws, region: us-west-2, cache_stopped_nodes: False}
auth: {ssh_user: ubuntu}

min_workers: 7
max_workers: 20
initial_workers: 7
autoscaling_mode: aggressive
# 
# Deep Learning AMI (Ubuntu) Version 21.0


head_node: {InstanceType: t3a.large, ImageId: ami-011238cd952065719}
#ray_address: 34.216.251.208
# Provider-specific config for worker nodes, e.g. instance type.
worker_nodes:
    InstanceType: t3a.large
    ImageId: ami-011238cd952065719 # my custom image

    # Run workers on spot by default. Comment this out to use on-demand.
    InstanceMarketOptions:
        MarketType: spot
        SpotOptions:
            MaxPrice: '0.025'  # Max Hourly Price
# in order to override the example-full.yaml we have to provide blank values here
setup_commands: # Set up each node.
    #- echo '. /home/ubuntu/anaconda3/etc/profile.d/conda.sh' >> ~/.bashrc
    #- echo 'conda activate tensorflow_p36' >> ~/.bashrc
    #- source ~/.bashrc
    #- pip install torch torchvision tabulate tensorboard 
    #- pip install -U https://s3-us-west-2.amazonaws.com/ray-wheels/latest/ray-0.9.0.dev0-cp36-cp36m-manylinux1_x86_64.whl
    #- pip install bayesian-optimization
    #- mkdir /home/ubuntu/baselines
    - echo 'Done.'
head_setup_commands:
    - pip install boto3==1.4.8  # 1.4.8 adds InstanceMarketOptions
worker_start_ray_commands:
    - ray stop
    #- ulimit -n 65536; ray start --address=54.201.36.207:6379 --object-manager-port=8076
    - ulimit -n 65536; ray start --address=$RAY_HEAD_IP:6379 --redis-password='5241590000000000' --object-manager-port=8076
    
file_mounts: {
#    "/home/ubuntu/baselines": "/home/ubuntu/lemmonw/flatlands/neurips2020-flatland-baselines",
#    "/path2/on/remote/machine": "/path2/on/local/machine",
}</pre>
<ul>
<li>after ray autoscaler generates ray_autoscaler key pair import it to cli
tool:
</li>
</ul>


<pre
class="prettyprint">ssh-keygen -y -f ~/.ssh/ray-autoscaler_us-west-2.pem > file:///home/ubuntu/.ssh/ray-autoscaler_us-west-2.pub
aws ec2 import-key-pair --key-name ray-autoscaler_us-west-2 --public-key-material ~/.ssh/ray-autoscaler_us-west-2.pub
</pre>
<ul>
<li>commands to start virtual x frame buffer, x11vnc, and chrome:
</li>
</ul>


<pre
class="prettyprint">nohup Xvfb :0 -ac -screen 0 1900x1000x24 &
sudo nohup /usr/bin/x11vnc -ncache 10 -ncache_cr -display :0 -forever -shared -logappend /var/log/x11vnc.log -bg -noipv6 -localhost > nohupxllvnc.out
DISPLAY=:0 chromium-browser &
# check the log file to make sure x11vnc was opened on port 5900
</pre>
<h3>Cluster image setup</h3>
<ul>
<li>I created a single image for both the head and workers. I use spot instances
to save $$.
<li>Request increase from default spot limit of 20 to whatever you need: <a
href="https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-ec2-spot-instances">https://console.aws.amazon.com/support/home#/case/create?issueType=service-limit-increase&limitType=service-code-ec2-spot-instances</a>
</li>
</ul>
<p>
<em>provision.sh</em>
</p>


<pre
class="prettyprint">sudo apt-get update
sudo apt-get install -y python3-pip
echo 'alias python=python3' >> ~/.bashrc
echo 'alias pip=pip3' >> ~/.bashrc
source ~/.bashrc
pip install --upgrade pip
sudo apt-get install -y libcairo2-dev libsm6 libxext6 libxrender-dev
pip install --no-input numpy scikit-learn torch torchvision tabulate tensorboard  bayesian-optimization opencv-python pycairo flatland-rl dm-tree ray[rllib]==0.8.5 pyhumps wandb lz4 tensorflow gputil
sudo apt-get install -y golang libjpeg-turbo8-dev make
pip install --no-input gym
git clone http://gitlab.aicrowd.com/flatland/neurips2020-flatland-baselines.git

# clear apt cache
sudo apt-get clean
sudo apt-get autoclean
sudo apt-get -y autoremove

# you might need to run these commands from sudo /bin/bash
[ -f /home/ubuntu/.ssh/authorized_keys ] && rm /home/ubuntu/.ssh/authorized_keys
find /var/log -type f | while read f; do echo -ne '' > $f; done

# clean other things
test -d /var/lib/cloud && /bin/rm -rf /var/lib/cloud/*
test -f /etc/udev/rules.d/70-persistent-net.rules && /bin/rm /etc/udev/rules.d/70-persistent-net.rules
# iirc in order for the newly created image to work properly, previous key material must be purged.
shred -u /etc/ssh/*_key /etc/ssh/*_key.pub
sudo shred ~/.ssh/*.pem
rm ~/.ssh/*.pem

# clear bash history
history -c
history -w
unset HISTFILE
[ -f /root/.bash_history ] && rm /root/.bash_history
[ -f /home/ubuntu/.bash_history ] && rm /home/ubuntu/.bash_history</pre>
<h3>neurips2020-flatland-baselines</h3>
<h4>Tips:</h4>
<p>
The train.py file expects to be called from its home directory. When using ray
submit config.yaml train.py, the file is executed from /home/ubuntu, which
breaks the project in several ways. To fix this, add the following to train.py:
</p>
<p>
<em>train.py</em>
</p>


<pre
class="prettyprint">import sys
# because ray starts at homedir, change path!
sys.path.append("/home/ubuntu/neurips2020-flatland-baselines")
# for load_envs to work, we have to either update the cwd to be baselinse basedir, or change working directory!
os.chdir("/home/ubuntu/neurips2020-flatland-baselines")</pre>
<h3>Starting Autoscaler </h3>
<h4>Tips:</h4>
<ol>
<li>When you run ray up yourconfig.yaml, ray will merge your config with its
default config, which is your
python/site-packages/ray/.../aws/example-full.yaml. <strong>If you don’t
explicitly override the setup_commands section, the default will upgrade ray to
the latest version immediately after booting the head and workers. </strong>So
if you want to keep the version of ray[rllib]==0.8.5, you have to override that
section.
<li>commands:
</li>
</ol>
<table>
  <tr>
   <td>exit conda environment
   </td>
   <td><code>conda deactivate</code>
   </td>
  </tr>
  <tr>
   <td>exit conda environment
   </td>
   <td><code>conda activate flatland-baseline-cpu-env</code>
   </td>
  </tr>
  <tr>
   <td>change directory
   </td>
   <td><code>cd lemmonw/flatlands/neurips2020-flatland-baselines/</code>
   </td>
  </tr>
  <tr>
   <td>start cluster
   </td>
   <td><code>yes | ray up tune-default.yaml</code>
   </td>
  </tr>
  <tr>
   <td>send local files to head box
   </td>
   <td><strong><code>ray rsync-up tune-default.yaml
/home/ubuntu/lemmonw/flatlands/neurips2020-flatland-baselines
/home/ubuntu/</code></strong>
   </td>
  </tr>
  <tr>
   <td>submit job on cluster
   </td>
   <td><strong><code>ray submit tune-default.yaml train.py --
--ray-address=$(headip):6379 -f
/home/ubuntu/neurips2020-flatland-baselines/baselines/action_masking_and_skipping/apex_tree_obs_small_v0.yaml<br>ray
submit tune-default.yaml mnist_pytorch_trainable.py --
--ray-address=$(headip):6379</code></strong>
   </td>
  </tr>
  <tr>
   <td>sometimes ray dashboard tune-default.yaml
<p>
 doesnt work. do this instead:
<ol>
<li>log into head node
<li>examine dashboard port
<li>ssh into head node, with reverse port forwarding of dashboard
</li>
</ol>
   </td>
   <td><code>ray attach config.yaml</code>
<p>
<code>ps aux | grep dashboard</code>
<p>
<code>ssh -i ~/.ssh/ray-autoscaler_us-west-2.pem -L
8265:localhost:<strong>8265</strong> ubuntu@$(headip)</code>
   </td>
  </tr>
  <tr>
   <td>view logs
   </td>
   <td><code>ray exec
/home/ubuntu/lemmonw/flatlands/neurips2020-flatland-baselines/tune-default.yaml
'tail -n 100 -f /tmp/ray/session_*/logs/monitor*'</code>
   </td>
  </tr>
  <tr>
   <td>other commands
   </td>
   <td><code>ray kill-random-node tune-default.yaml --hard</code>
<p>
<code><em>Analyze your results on TensorBoard by starting TensorBoard on the
remote head machine.</em></code>
<code><em># Go to http://localhost:6006
to access TensorBoard.</em></code>
<code>ray exec tune-default.yaml
'tensorboard --logdir=~/ray_results/ --port 6006' --port-forward 6006</code>
<p>
<code>ray exec CLUSTER.YAML 'jupyter lab --port 6006' --port-forward 6006</code>
<p>
<code>ray exec CLUSTER.YAML 'tune ls ~/ray_results'</code>
<p>
<code>ray rsync-up CLUSTER.YAML</code>
<p>
<code>ray rsync-up tune-default.yaml
/home/ubuntu/lemmonw/flatlands/neurips2020-flatland-baselines/ /home/ubuntu/ #
this will copy all to home directory</code>
<p>
<strong><code> </code></strong>
<code>ray rsync-down CLUSTER.YAML
'~/ray_results' ~/cluster_results</code>
<p>
<code>ray down CLUSTER.YAML [-y]</code>
<p>
<code>ray start --head --redis-port 7778</code>
<p>
<code>redis-cli -p 7778 --pass 5241590000000000 ping</code>
<p>
<code>redis-cli -p 6379 -a 5241590000000000 ping</code>
<p>
<code>localhost:8268?ray dashboard</code>
<p>
<code> </code>
<p>
<code>conda env create -f environment-gpu.yml</code>
<p>
<code>ray exec-cmd cluster.yaml "cd /home/ubuntu/baselines; python train.py
--ray-redis-max-memory=52428800 --ray-memory=52428800 --ray-num-cpus=2
--queue-trials -f
baselines/global_density_obs/sparse_small_apex_expdecay_maxt1000.yaml"</code>
   </td>
  </tr>
</table>
