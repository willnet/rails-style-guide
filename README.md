# rails-style-guide

railsのスタイルガイド

## skinny Controller, fat Model

### controller の action を分割する

UsersController があるとします。さらに、管理画面において、ユーザの名前を更新するフォームと、メールアドレスを更新するためのフォームを別のURLで提供する必要があるとします。簡単に書くとこのようになるでしょうか。

```ruby
class Admin::UsersController < ApplicationController
  before_action :authenticate

  def index
    @users = User.per(10).page(params[:page])
  end

  def show
    @user = User.find(params[:id])    
  end

  def edit_name
    @user = User.find(params[:id])    
  end

  def update_name
    @user = User.find(params[:id])    
    unless @user.update(name: params[:user][:name])
      render :edit_name
    end
  end

  def edit_email
    @user = User.find(params[:id])
  end

  def update_email
    @user = User.find(params[:id])
    unless @user.update(name: params[:user][:email])
      render :edit_email
    end    
  end    
end
```

しかし上記のように、基本の7つ(index, show, new, create, edit, update, destroy)以外のアクションを定義するのは良くない傾向です。これはまだ十分小さいコードですが、この調子でなんでも一つのコントローラに詰め込もうとするとコントローラのコードが肥大化し、内容を理解するのが大変になります。

それではどのようにすると良いでしょうか。コントローラを適切に分割します。

```ruby
class Admin::UsersController < ApplicationController
  before_action :authenticate

  def index
    @users = User.per(10).page(params[:page])
  end

  def show
    @user = User.find(params[:id])        
  end
end

class Admin::Users::NamesController < ApplicationController
  before_action :authenticate

  def edit
    @user = User.find(params[:id])
  end

  def update
    @user = User.find(params[:id])    
    unless @user.update(name: params[:user][:name])
      render :edit
    end
  end
end

class Admin::Users::EmailsController < ApplicationController
  before_action :authenticate

  def edit
    @user = User.find(params[:id])        
  end

  def update
    @user = User.find(params[:id])            
    unless @user.update(name: params[:user][:email])
      render :edit
    end
  end
end
```

リソースごとに分割することで、基本の7つのアクションの範囲内で書くことができました。


## skinny ActiveRecord Model
