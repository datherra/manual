```ruby
# encoding: utf-8
require "spec_helper"

describe "Add question" do
  context "when providing valid data" do
    let(:user) { users(:john) }
    let(:category) { categories(:rails) }

    before do
      login_with(user)

      visit "/"
      click_link "Adicionar pergunta"

      fill_in "Título da pergunta", :with => "Title"
      fill_in "Descreva sua pergunta", :with => "Description"
      select category.name, :from => "Categoria"
      click_button "Adicionar pergunta"
    end

    it "redirects to the question page" do
      current_path.should match(%r[/questions/\d+])
    end

    it "displays success message" do
      page.should have_content("Pergunta 'Title' foi criada com sucesso!")
    end
  end

  context "when providing invalid data" do
    let(:user) { users(:john) }
    let(:category) { categories(:rails) }

    before do
      login_with(user)

      visit "/"
      click_link "Adicionar pergunta"
      click_button "Adicionar pergunta"
    end

    it "renders form page" do
      current_path.should eql(questions_path)
    end

    it "displays error messages" do
      page.should have_form_errors
    end
  end

  context "when unlogged" do
    before do
      visit "/"
      click_link "Adicionar pergunta"
    end

    it "redirects to the login page" do
      current_path.should eql(login_path)
    end
  end
end

```