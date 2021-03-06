# Lesson-6 いいねとコメントの非同期実装
<details>
<summary>通常実装までをクリックで展開</summary>

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
        @book = Book.find(params[:book_id])
        favorite = current_user.favorites.build(book_id: params[:book_id])
        favorite.save
        redirect_to request.referer
    end
    
    def destroy
        @book = Book.find(params[:book_id])
        favorite = current_user.favorites.find_by(book_id: params[:book_id], user_id: current_user.id)
        favorite.destroy
        redirect_to request.referer
    end
    
end
```
</details>

### ①コントローラーの変更
#### ①favoritesコントローラーの変更
```
class FavoritesController < ApplicationController
    
    def create
        @book = Book.find(params[:book_id])
        favorite = current_user.favorites.build(book_id: params[:book_id])
        favorite.save
    end
    
    def destroy
        @book = Book.find(params[:book_id])
        favorite = current_user.favorites.find_by(book_id: params[:book_id], user_id: current_user.id)
        favorite.destroy
    end
    
end

```
#### ②book_commentsコントローラーの変更
```
class BookCommentsController < ApplicationController
    
    def create
        @book = Book.find(params[:book_id])
        @comment = current_user.book_comments.new(book_comment_params)
        @comment.book_id = @book.id
        @comment.save
    end
    
    def destroy
        @book = Book.find(params[:book_id])
        @comment = BookComment.find_by(id: params[:id], book_id: params[:book_id])
        @comment.destroy
    end
    
    private

    def book_comment_params
        params.require(:book_comment).permit(:comment)
    end
    
end
```
### ②viewのテンプレート可および変更
#### ①_fav.html.viewの変更
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
#### ②book_comments/_index.html.erbの作成
```
      <table class='table'>
        <t-body>
          <% book.book_comments.each do |book_comment| %>
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
                      <%= link_to "Destroy", book_book_comment_path(book_comment.book, book_comment), method: :delete, remote: true, class: "btn btn-sm btn-danger destroy_book_#{@book.id}" %>
                      </div>
                  <% end %>
                    </td>
            <% end %></tr>
          <% end %>
        </t-body>
      </table>
```
#### ③book_comments/_form.html.erbの作成
```
        <%= form_with(model:[book, comment], remote: true) do |f| %>
          <%= f.text_area :comment, rows:'3',placeholder: "コメントをここに", :style=>"width:100%;" %>
          <%= f.submit "送信する" %>
        <% end %>
```
#### ④book_comments/_count.html.erbの作成
```
コメント件数：<%= book.book_comments.count %>
```
#### ⑤book_comments/_error.html.erbの作成
```
          <% if comment.errors.any? %>
              <ul>
                <% comment.errors.full_messages.each do |message| %>
                <li><%= message %></li>
                <% end %>
              </ul>
          <% end %>
```
### ③_indexとshowのviewを変更
books/show.html.erb
```
<div class='container'>
  <div class='row'>
    <div class='col-md-3'>
      <h2>User info</h2>
      <%= render 'users/info', user: @book.user %>
      <h2 class="mt-3">New book</h2>
      <%= render 'books/form', book: Book.new %>
    </div>
    <div class='col-md-8 offset-md-1'>
  		<h2>Book detail</h2>
  		<table class='table'>
  		  <tr>
  		    <td><%= link_to user_path(@book.user) do %>
            <%= attachment_image_tag(@book.user, :profile_image, :fill, 100, 100, fallback: "no-image-icon.jpg") %><br>
            <%= @book.user.name %>
            <% end %>
          </td>
          <td><%= link_to @book.title, book_path(@book) %></td>
          <td><%= @book.body %></td>
          <td id="comment-count"><%= render 'book_comments/count', book: @book %></td>
          <td id="favs_buttons_<%= @book.id %>"><%= render 'favorites/fav', book: @book %></td>
          <td>
            <% if @book.user == current_user %>
              <%= link_to 'Edit', edit_book_path(@book), class: "btn btn-sm btn-success edit_book_#{@book.id}" %>
            <% end %>
          </td>
          <td>
            <% if @book.user == current_user %>
              <%= link_to 'Destroy', book_path(@book), method: :delete, data: { confirm: '本当に消しますか？' }, class: "btn btn-sm btn-danger destroy_book_#{@book.id}"%>
            <% end %>
          </td>
        </tr>
      </table>
      <div id='comments'><%= render 'book_comments/index', {book: @book} %></div>
      <div id="error_explanation"><%= render 'book_comments/error', comment: @comment %></div>
      <div id="new-comment"><%= render 'book_comments/form', {book: @book, comment: @comment} %></div>
  </div>
</div>
```
books/_index.html.erb
```
<table class='table table-hover table-inverse'>
  <thead>
    <tr>
      <th></th>
      <th>Title</th>
      <th>Opinion</th>
      <th colspan="3"></th>
    </tr>
  </thead>
  <tbody>
    <% books.each do |book| %>
      <tr>
        <td><%= link_to user_path(book.user) do %>
          <%= attachment_image_tag(book.user, :profile_image, :fill, 50, 50, fallback: "no-image-icon.jpg") %>
          <% end %>
        </td>
        <td><%= link_to book.title, book_path(book), class: "book_#{book.id}" %></td>
        <td><%= book.body %></td>
        <td id="comment-count"><%= render 'book_comments/count', book: book %></td></td>
        <td id="favs_buttons_<%= book.id %>"><%= render 'favorites/fav', book: book %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```
### ④jqueryの記述
#### ①book_comments/create.js.erb
```
jQuery("#comment-count").html("<%= j(render 'book_comments/count', { book: @book }) %>");
jQuery("#comments").html("<%= j(render 'book_comments/index', { book: @book }) %>");
jQuery("#error_explanation").html("<%= j(render 'book_comments/error', { comment: @comment }) %>");
jQuery("#book_comment_comment").val("");
```
#### ②book_comments/destroy.js.erb
```
jQuery("#comments").html("<%= j(render 'book_comments/index', { book: @book }) %>");
jQuery("#comment-count").html("<%= j(render 'book_comments/count', { book: @book }) %>");
```
#### ③favorites/create.js.erb
```
$('#favs_buttons_<%= @book.id %>').html("<%= j(render partial: 'favorites/fav', locals: {book: @book}) %>");
```
#### ④favorites/destroy.js.erb
```
$('#favs_buttons_<%= @book.id %>').html("<%= j(render partial: 'favorites/fav', locals: {book: @book}) %>");
```
