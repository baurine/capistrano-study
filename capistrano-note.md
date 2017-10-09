# Capistrano Study

**Resources:**

- [Capistrano](http://capistranorb.com/)
- [Capistrano README](https://github.com/capistrano/capistrano/blob/master/README.md)
- [自动化部署工具 Capistrano 与 Mina](http://blog.fir.im/cap_and_mina/)
- [Rails 中自动布署工具 mina 的经验谈](http://yafeilee.me/blogs/82)

Capistrano: A deployment automation tool built on Ruby, Rake, and SSH.

简单地说，Capistrano 基于 Rake 并扩展了 Rake 的 DSL，通过 SSH 自动登录远程服务器并帮你执行各种脚本。

## Note 1

Capistrano 在非 Rails 项目中的使用。

### Step 1 - Install Capistrano

在目录上创建 Gemfile 文件 (如果是 Rails 项目则已存在)，添加对 capistrano gem 的依赖：

    # Gemfile
    group :development do
      gem "capistrano", "~> 3.9"
    end

然后在此目录下执行 `bundle` 安装所有依赖。安装好 capistrano，你就可以在命令行中执行 `cap` 命令了。

### Step 2 - Initial Capistrano Configuration

在项目目录下执行：

    $ bundle exec cap install
    # 或者简写成
    $ cap install [STAGES=staging,production]

将生成 capistrano 的模板代码，包括以下文件：

    ├── Capfile
    ├── config
    │   ├── deploy
    │   │   ├── production.rb
    │   │   └── staging.rb
    │   └── deploy.rb
    └── lib
        └── capistrano
                └── tasks

默认只生成了部署到 staging 和 produtcion 的配置，如果还要生成其它 stages，则在执行 `cap install` 时添加 `STAGES=xxx,yyy` 参数。

Capfile 描述了在此目录下执行 cap 时，要加载的其它 tasks，插件，以及在 lib/capistrano/tasks 下的 tasks 等。可以把一些公用的 task 放到 lib/capistrano/tasks 下。

config 目录下则是部署到各个 stage 的 tasks。重点是学习 config 目录下的 tasks 怎么写。

Capistrano 默认已经带了很多常用的 tasks，用 `cap -T` 可以查看这些已有的 tasks：

    $ cap -T
    cap deploy                         # Deploy a new release
    cap deploy:check                   # Check required files and directories exist
    cap deploy:check:directories       # Check shared and release directories exist
    cap deploy:check:linked_dirs       # Check directories to be linked exist in shared
    cap deploy:check:linked_files      # Check files to be linked exist in shared
    cap deploy:check:make_linked_dirs  # Check directories of files to be linked exist in shared
    cap deploy:cleanup                 # Clean up old releases
    cap deploy:cleanup_rollback        # Remove and archive rolled-back release
    cap deploy:finished                # Finished
    cap deploy:finishing               # Finish the deployment, clean up server(s)
    cap deploy:finishing_rollback      # Finish the rollback, clean up server(s)
    cap deploy:log_revision            # Log details of the deploy
    cap deploy:published               # Published
    cap deploy:publishing              # Publish the release
    cap deploy:revert_release          # Revert to previous release timestamp
    cap deploy:reverted                # Reverted
    cap deploy:reverting               # Revert server(s) to previous release
    cap deploy:rollback                # Rollback to previous release
    cap deploy:set_current_revision    # Place a REVISION file with the current revision SHA in the current release path
    cap deploy:started                 # Started
    cap deploy:starting                # Start a deployment, make sure server(s) ready
    cap deploy:symlink:linked_dirs     # Symlink linked directories
    cap deploy:symlink:linked_files    # Symlink linked files
    cap deploy:symlink:release         # Symlink release to current
    cap deploy:symlink:shared          # Symlink files and directories from shared to release
    cap deploy:updated                 # Updated
    cap deploy:updating                # Update server(s) by setting up a new release
    cap doctor                         # Display a Capistrano troubleshooting report (all doctor: tasks)
    cap doctor:environment             # Display Ruby environment details
    cap doctor:gems                    # Display Capistrano gem versions
    cap doctor:servers                 # Display the effective servers configuration
    cap doctor:variables               # Display the values of all Capistrano variables
    cap git:check                      # Check that the repository is reachable
    cap git:clone                      # Clone the repo to the cache
    cap git:create_release             # Copy repo to releases
    cap git:set_current_revision       # Determine the revision that will be deployed
    cap git:update                     # Update the repo mirror to reflect the origin state
    cap git:wrapper                    # Upload the git wrapper script, this script guarantees that we can script git without getting an inte...
    cap install                        # Install Capistrano, cap install STAGES=staging,production

其它一些常用命令：

    # list all available tasks
    $ bundle exec cap -T

    # deploy to the staging environment
    $ bundle exec cap staging deploy

    # deploy to the production environment
    $ bundle exec cap production deploy

    # simulate deploying to the production environment
    # does not actually do anything
    $ bundle exec cap production deploy --dry-run

    # list task dependencies
    $ bundle exec cap production deploy --prereqs

    # trace through task invocations
    $ bundle exec cap production deploy --trace

    # lists all config variable before deployment tasks
    $ bundle exec cap production deploy --print-config-variables

### 部署后的服务器上的目录结构

[Structure](http://capistranorb.com/documentation/getting-started/structure/)

假设在 config/deploy.rb 中指定 :deploy_to 的值为如下所示：

    # config/deploy.rb
    set :deploy_to, '/var/www/my_app_name'

那么部署到服务上后，在 `/var/www/my_app_name` 目录中的结构：

    ├── current -> /var/www/my_app_name/releases/20150120114500/
    ├── releases
    │   ├── 20150080072500
    │   ├── 20150090083000
    │   ├── 20150100093500
    │   ├── 20150110104000
    │   └── 20150120114500
    ├── repo
    │   └── <VCS related data>
    ├── revisions.log
    └── shared
        └── <linked_files and linked_dirs>

### 配置

[Configuration](http://capistranorb.com/documentation/getting-started/configuration/)

**配置文件：**

- 全局：config/deploy.rb
- stage：config/deploy/stage_name.rb

**访问变量**

用 set 方法赋值：

(好奇，为什么不直接用 = 赋值?)

    set :application, 'MyLittleApplication'

    # use a lambda to delay evaluation
    set :special_thing, -> { "SomeThing_#{fetch :other_config}" }

用 fetch 方法取值：

    fetch :application
    # => "MyLittleApplication"

    fetch(:special_thing, 'some_default_value')
    # will return the value if set, or the second argument as default value

从 3.5 开始，可以用 append 和 remove 添加值到数组或从数组移除值：

    append :linked_dirs, ".bundle", "tmp"
    remove :linked_dirs, ".bundle", "tmp"

**可赋值的变量**

略。有点多，需要时再仔细看。

重要的几个：

- :application
- :deploy_to
- :repo_url

### 用户输入

[User Input](http://capistranorb.com/documentation/getting-started/user-input/)

使用 ask 方法向用户请求输入。但注意的是，ask 并不会马上执行 (相当于只是声明或注册了一个行为)，只有等到 fetch 这个变量的值时，这个 ask 方法才会触发。

    # used in a configuration
    ask(:database_name, "default_database_name")

    # used in a task
    desc "Ask about breakfast"
    task :breakfast do
      ask(:breakfast, "pancakes")
      on roles(:all) do |h|
        execute "echo \"$(whoami) wants #{fetch(:breakfast)} for breakfast!\""
      end
    end

上例中，`ask(:breakfast, "pancakes")` 要等到执行 `fetch(:breakfast)` 时才会触发。如果没有 fetch，就永远不会触发。

ask 的第二个参数是默认值，还有第三个参数 echo，如果 echo 设置为 false，那么你的输入不会显示在屏幕上，在输密码时有用。

    ask(:database_password, 'default_password', echo: false)

### Preparing Your Application

[Preparing Your Application](http://capistranorb.com/documentation/getting-started/preparing-your-application/)

1. 提交代码到部署服务器可访问的仓库
1. 从仓库中移除敏感文件，比如 database.yml
1. 使用 `cap install` 初始化 capinstrano 的配置
1. 在 config/deploy/stage.rb 中设置各 stage 不同的配置 (比如服务器地址，部署的分支名)
1. 在 config/deploy.rb 中设置共享的配置

如果各个 stage 使用相同的服务器，那么可以配置在 config/deploy.rb 中，否则，配置到各 stage 自己的 config/deploy/stage.rb 中。

实际项目中，各 stage 应该是使用不同的服务器，或是同一名服务器上的不同域名。

**配置 config/deploy/stage.rb**

主要是配置 role 和 server，分别使用 role 方法和 server 方法。实际两个方法的作用是相似的，只是表现形式不太一样。一个为 role 为主，一个以 server 为主。如果你要为同一个 role 配置多个 server，用前者，如果要为一个 server 配置多个 role，用后者。

以下两种配置是等价的。

    # using simple syntax
    role :web, %w{hello@world.com example.com:1234}

    # using extended syntax (which is equivalent)
    server 'world.com', roles: [:web], user: 'hello'
    server 'example.com', roles: [:web], port: 1234

role 默认有三个：web，app，db。

[What exactly is a "role" in Capistrano?](https://stackoverflow.com/questions/1155218/what-exactly-is-a-role-in-capistrano)

明白了，role 是对服务器的分类，如果部署在多台服务器上才能体现出它的效果。假设有一个大项目，部署在多台服务器上，一些服务器部署的是数据库部分，而且数据库还分主从服务器，一些服务器部署的是代码部分。那么你就可以将这些运行数据库的服务器指定为 db role，将运行代码的服务器指定为 app role。然后你就可以指定一些 tasks，只在 db 服务器上执行，指定一些 tasks，只在 app 服务器上的执行，或是在全部类型的服务器上都执行，甚至可以指定一些 tasks 不在任何远程服务器上执行，只在本地执行。

本项目中，因为我们只有一台服务器，所以所有服务都跑在一台服务器上，包括代码和数据库，所以这台服务器的类型既是 app，又是 db。但是实际如果项目只有一台服务器上，完全可以不需要这么多 roles，任选一个就行了，因为没有什么意义。

    # config/deploy.rb
    server "my-ec2", roles: [:web, :app, :db]

而下面这个配置：

    role :web, %w{hello@world.com example.com:1234}

意思是说，作为 web role 的服务器有两台。

因此，下面这个配置：

    task :migrate, :roles => :db, :only => { :primary => true } do
      # ...
    end

意思是说，migrate 这个 task，将只在 role 为 db 类型，且是主 db 服务器上执行，在其它类型的服务器上不执行。

**配置 config/deploy.rb**

设置 :application, :repo_url, :branch 等变量。

    set :application, 'rails3-bootstrap-devise-cancan-demo'
    set :repo_url, 'https://github.com/capistrano/rails3-bootstrap-devise-cancan'
    set :branch, 'master'

截止到目前为止，capistrano 知道怎么找到部署服务器和去哪里下载代码了。

### Flow

[Flow](http://capistranorb.com/documentation/getting-started/flow/)

执行 `cap production deploy` 后的整个流程：

    deploy:starting    - start a deployment, make sure everything is ready
    deploy:started     - started hook (for custom tasks)
    deploy:updating    - update server(s) with a new release
    deploy:updated     - updated hook
    deploy:publishing  - publish the new release
    deploy:published   - published hook
    deploy:finishing   - finish the deployment, clean up everything
    deploy:finished    - finished hook

其中 hook 的部分表示，这些 tasks 是可以自己定义的，在 deploy.rb 中通过 before 或 after 方法来在整个流程中插入自己定义的 tasks。加载的第三方插件也是通过 before 和 after 来进行 hook。

其余暂略，需要时再详细看。

### Rollbacks

暂略。

### Cold Start

[Cold Start](http://capistranorb.com/documentation/getting-started/cold-start/)

这一小节主要讲两个事情，一个是检测我们配置的服务器是否真的可访问，二是我们配置的 git repo 是不是真的可访问。

首先在 config/deploy.rb 中配置好部署服务器和 git repo：

    # config/deploy.rb

    # my-ec2 is already configured in ~/.ssh/config
    server "my-ec2", roles: [:web, :app, :db]
    set :user,        "ubuntu"

    set :application, "capistrano-study"
    set :repo_url,    "git@github.com:baurine/#{fetch(:application)}.git"

    set :deploy_to,   "/home/#{fetch(:user)}/apps/#{fetch(:application)}"

my-ec2 是我的 EC2 服务器，ubuntu 是可 ssh 登录的用户。

我们写一个 task 来检测服务器是否可访问，放在 lib/capistrano/tasks/access_check.rake 中：

    # lib/capistrano/tasks/access_check.rake
    desc "Check that we can access everything"
    task :check_write_permissions do
      on roles(:all) do |host|
        if test("[ -w #{fetch(:deploy_to)} ]")
          info "#{fetch(:deploy_to)} is writable on #{host}"
        else
          error "#{fetch(:deploy_to)} is not writable on #{host}"
        end
      end
    end

然后执行：

    $ cap staging check_write_permissions
    00:00 check_write_permissions
      /home/ubuntu/apps/capistrano-study is writable on my-ec2

(注意，/home/ubuntu/apps/capistrano-study 这个目录要先提前手动创建好，不然上面的 task 会执行失败。)

上面的 task 中，desc() 和 task() 方法是来自 Rake 的，还记得 Capistrano 是基于 Rake 的扩展吗？on()、roles()、test()、inf()、error() 方法来自 SSHKit。

检测 git repo 是否可访问，capistrano 有一个内置的 task 来检测：

    $ cap staging git:check
    00:00 git:wrapper
          01 mkdir -p /tmp
        ✔ 01 my-ec2 5.107s
          Uploading /tmp/git-ssh-capistrano-study-staging-baurine.sh 100.0%
          02 chmod 700 /tmp/git-ssh-capistrano-study-staging-baurine.sh
        ✔ 02 my-ec2 0.612s
    00:08 git:check
          01 git ls-remote git@github.com:baurine/capistrano-study.git HEAD
          01 13b3d2e6669fc22ac0a8dd35cfca57ad7ff96183	HEAD
        ✔ 01 my-ec2 1.610s

### Tasks

[Tasks](http://capistranorb.com/documentation/getting-started/tasks/)

一个自定义的 task 示例：

    # config/deploy.rb
    server 'example.com', roles: [:web, :app]
    server 'example.org', roles: [:db, :workers]

    desc "Report Uptimes"
    task :uptime do
      on roles(:all) do |host|
        within release_path do
          execute :any_commandiiet , "with args", :here, "and here"
          info "Host #{host} (#{host.roles.to_a.join(', ')}):\t#{capture(:uptime)}"
        end
      end
    end

within() 方法，参数是一个路径，表示进入到这个路径中，再执行 block 中的操作，block 结束时，回到原来的路径中。

execute() 方法，执行 shell 命令。文档里说 `execute(:bundle, :install)` 和 `execute('bundle install')` 的行为并不完全相同，但暂时可忽略。

### Local Tasks

[Local Tasks](http://capistranorb.com/documentation/getting-started/local-tasks/)

(我的理解，local task 是在本地执行的 task，并不会在任何远程服务器上执行，因为并没有给它指定任何 role。)

前面的 task 用 `on roles(:all)` 指定在哪些远程服务器上执行，如果把这个语句替换成 `run_locally`，则意味着这个 task 只在本地执行，而不是在远程服务器上执行。(这样理解应该没错吧。)

示例代码：

    desc 'Notify service of deployment'
    task :notify do
      run_locally do
        with rails_env: :development do
          rake 'service:notify'
        end
      end
    end

with() 方法设置运行时的环境变量。

简化写法：

    task :notify do
      %x('RAILS_ENV=development bundle exec rake "service:notify"')
    end
    # 或者
    task :notify do
      sh 'RAILS_ENV=development bundle exec rake "service:notify"'
    end

### Before / After Hooks

[Before / After Hooks](http://capistranorb.com/documentation/getting-started/before-after/)

Before / After 的使用的两种方式：

    # call an existing task
    before :starting, :ensure_user
    after :finishing, :notify

    # or define in block
    before :starting, :ensure_user do
      #
    end
    after :finishing, :notify do
      #
    end

在 Capistrano 中也可以像在 Rake 中使用依赖：

    desc "Create Important File"
    file 'important.txt' do |t|
      sh "touch #{t.name}"
    end
    desc "Upload Important File"
    task :upload => 'important.txt' do |t|
      on roles(:all) do
        upload!(t.prerequisites.first, '/tmp')
      end
    end

在 Capistrano 的 task 中使用 invoke() 方法调用其它 task。(在 Rake 中是使用 Rake::Task['task_name'].invoke。)

    namespace :example do
      task :one do
        on roles(:all) { info "One" }
      end
      task :two do
        invoke "example:one"
        on roles(:all) { info "Two" }
      end
    end

## Note 2

Capistrano 在 Rails 项目中的使用。

在 Rails 项目中可以使用 `capistrano-rails` 这个 gem，它依赖了 capistrano，在它的基础上增加了一些适用于 Rails 项目的 tasks，包括：

- Asset Pipeline Support
- Database Migration Support

安装：

    # Gemfile
    group :development do
      gem 'capistrano-rails', '~> 1.1'
    end

然后在 Capfile 中导入对 'capistrano/rails'：

    # Capfile
    # ...
    require 'capistrano/deploy'
    require 'capistrano/rails'
