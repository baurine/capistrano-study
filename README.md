# Capistrano Study

[Study Note](./capistrano-note.md)

本项目展示了一个最简单的可成功部署的 Capistrano 的配置。

前提条件，你需要一台服务器，本项目使用的是 EC2。

[config/deploy.rb](./config/deploy.rb) 的内容：

    # my-ec2 is already configured in ~/.ssh/config
    server "my-ec2", roles: [:web, :app, :db]
    # ubuntu is an username in my-ec2 server
    set :user,        "ubuntu"

    set :application, "capistrano-study"
    set :repo_url,    "git@github.com:baurine/#{fetch(:application)}.git"
    set :deploy_to,   "/home/#{fetch(:user)}/apps/#{fetch(:application)}"

    after 'deploy:published', 'deploy:restart'

    namespace :deploy do
      task :restart do
        on roles(:app) do
          within release_path do
            # 注意，是 './test-command.sh'，不能是 'test-command.sh'
            execute './test-command.sh'
          end
        end
      end
    end

执行 `cap production deploy` 后的 log：

    > cap production deploy
    00:00 git:wrapper
          01 mkdir -p /tmp
        ✔ 01 my-ec2 3.786s
          Uploading /tmp/git-ssh-capistrano-study-production-baurine.sh 100.0%
          02 chmod 700 /tmp/git-ssh-capistrano-study-production-baurine.sh
        ✔ 02 my-ec2 1.024s
    00:07 git:check
          01 git ls-remote git@github.com:baurine/capistrano-study.git HEAD
          01 0db72ef7b31550735d3152fa0c1c4dd56deac464	HEAD
        ✔ 01 my-ec2 2.250s
    00:09 deploy:check:directories
          01 mkdir -p /home/ubuntu/apps/capistrano-study/shared /home/ubuntu/apps/capistrano-study/releases
        ✔ 01 my-ec2 1.327s
    00:15 git:clone
          The repository mirror is at /home/ubuntu/apps/capistrano-study/repo
    00:16 git:update
          01 git remote set-url origin git@github.com:baurine/capistrano-study.git
        ✔ 01 my-ec2 0.584s
          02 git remote update --prune
          02 Fetching origin
          02 From github.com:baurine/capistrano-study
          02    714288a..0db72ef  master     -> master
        ✔ 02 my-ec2 2.794s
    00:20 git:create_release
          01 mkdir -p /home/ubuntu/apps/capistrano-study/releases/20171006142935
        ✔ 01 my-ec2 1.534s
          02 git archive master | /usr/bin/env tar -x -f - -C /home/ubuntu/apps/capistrano-study/releases/20171006142935
        ✔ 02 my-ec2 3.075s
    00:33 deploy:set_current_revision
          01 echo "0db72ef7b31550735d3152fa0c1c4dd56deac464" >> REVISION
        ✔ 01 my-ec2 1.116s
    00:34 deploy:symlink:release
          01 ln -s /home/ubuntu/apps/capistrano-study/releases/20171006142935 /home/ubuntu/apps/capistrano-study/releases/current
        ✔ 01 my-ec2 1.078s
          02 mv /home/ubuntu/apps/capistrano-study/releases/current /home/ubuntu/apps/capistrano-study
        ✔ 02 my-ec2 1.782s
    00:39 deploy:restart
          01 ./test-command.sh
          01 hello capistro
        ✔ 01 my-ec2 0.922s
    00:41 deploy:log_revision
          01 echo "Branch master (at 0db72ef7b31550735d3152fa0c1c4dd56deac464) deployed as release 20171006142935 by baurine" >> /home/ubuntu/app…
        ✔ 01 my-ec2 0.717s

