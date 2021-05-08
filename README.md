# Lesson-3 いいねとコメント

## コメント機能

### ①モデル作成
```
rails g model BookComment comment:text user_id:integer book_id:integer
```
### ②モデルの関連付けとバリデーション
#### user.rbとbook.rbに関連付け

```
has_many :book_comments, dependent: :destroy
```
#### book_comment.rbに関連付けとバリデーション
```
belongs_to :user
belongs_to :book
validates :comment, presence: true
```
### ③コメント用のコントローラーを作成
```
rails g controller BookComments
```
createとdestroyアクションを作成

### ④ルーティングの追記
```
resources :books
```
を以下に変更
```
  resources :books do
    resources :book_comments, only: [:create, :destroy]
  end
```  
### ⑤ビューの作成
#### ①showに配置する一覧とフォームのview
    
      <div class='comments'>
      <table class='table'>
        <t-body>
          <% @book.book_comments.each do |book_comment| %>
            <tr><% unless book_comment.id.nil? %>
                <td>
                  <%= link_to user_path(book_comment.user) do %>
                    <%= attachment_image_tag(book_comment.user, :profile_image, :fill, 100, 100, fallback: "no-image-icon.jpg") %><br>
                    <%= book_comment.user.name %>
                  <% end %>
                </td>
                <td><%= book_comment.comment %></td>
                <td>
                  <% if book_comment.user == current_user %>
                    <div class="comment-delete">
                      <%= link_to "Destroy", book_book_comment_path(book_comment.book, book_comment), method: :delete, class: "btn btn-sm btn-danger destroy_book_#{@book.id}" %>
                      </div>
                  <% end %>
                    </td>
            <% end %></tr>
          <% end %>
        </t-body>
      </table>
      </div>
      <div class="new-comment">
        <%= form_with(model:[@book, @comment], local: true) do |f| %>
          <% if @comment.errors.any? %>
            <div id="error_explanation">
              <ul>
                <% @comment.errors.full_messages.each do |message| %>
                <li><%= message %></li>
                <% end %>
              </ul>
            </div>
          <% end %>
          <%= f.text_area :comment, rows:'3',placeholder: "コメントをここに", :style=>"width:100%;" %>
          <%= f.submit "送信する" %>
        <% end %>
      </div>
    
#### ②indexに配置するカウントview
```
<td>コメント件数：<%= @book.book_comments.count %></td>
```
### ⑥コメントのインスタンス変数を記述
 books_controller.rbのshowに
 ```
 @comment = BookComment.new
 ```
 を追加
### ⑦コメントコントローラーアクションを記述
```
class BookCommentsController < ApplicationController
    
    def create
        book = Book.find(params[:book_id])
        comment = current_user.book_comments.new(book_comment_params)
        comment.book_id = book.id
        comment.save
        redirect_to book_path(book)
        もしくは redirect_to request.referer
    end
    
    def destroy
        BookComment.find_by(id: params[:id], book_id: params[:book_id]).destroy
        redirect_to book_path(params[:book_id])
        もしくは redirect_to request.referer
    end
    
    private

    def book_comment_params
        params.require(:book_comment).permit(:comment)
    end
    
end
```
## いいね機能
### ①ルーティングの追加
```
  resources :post_images, only: [:new, :create, :index, :show, :destroy] do
    resource :favorites, only: [:create, :destroy] 
  end
```
### ②モデル作成
```
rails g model Favorite user_id:integer book_id:integer
```
### ③アソシエーションの追加
#### ①bookモデルに追加
```
  has_many :favorites, dependent: :destroy
  def favorited_by?(user)
    favorites.where(user_id: user.id).exists?
  end
```
#### ②favoriteモデルに追加
```
  belongs_to :user
  belongs_to :book
```
### ④favoritesコントローラーを作成
```
rails g controller Favorites
```
### ⑤viewをテンプレートで作成
_fav.html.erb
```
<% if book.favorited_by?(current_user) %>
    <%= link_to book_favorites_path(book), :style=>"color:red;", method: :delete, remote: true do %>
        ♥<%= book.favorites.count %>
    <% end %>
<% else %>
    <%= link_to book_favorites_path(book), :style=>"color:blue;", method: :post, remote: true do %>
        ♥<%= book.favorites.count %>
    <% end %>
<% end %>
```
showだと
```
<%= render 'favorites/fav', book: @book %>
```
indexだと
```
<%= render 'favorites/fav', book: book %>
```
### ⑥favoriteコントローラーにアクションを記載
```
class FavoritesController < ApplicationController
  def create
    book = Book.find(params[:book_id])
    favorite = current_user.favorites.new(book_id: book.id)
    favorite.save
    redirect_to request.referer
  end

  def destroy
    book = Book.find(params[:book_id])
    favorite = current_user.favorites.find_by(book_id: book.id)
    favorite.destroy
    redirect_to request.referer
  end
end
```