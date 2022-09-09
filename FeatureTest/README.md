# Feature Tests

Mục đích của Feature Test (Integration Test) là để chắc chắn rằng tất cả các components làm việc với nhau như ta mong muốn.

## Guide
Feature test PHẢI cover được TOÀN BỘ route của application, và phải tuân thủ các mục đích sau:

**HTTP**
- Authentication và Policy tests. Mỗi test case cần có những test assertion riêng biệt.
- Kiểm tra Status Codes cho từng loại response.
- Kiểm tra Redirect Codes và Paths cho các các sự kiện khác nhau.
- Kiểm tra tính đúng đắn của JSON responses.
- Kiểm tra Error Handlers cho các response.
- Kiểm tra tính đúng đắn của API versions (nếu cần thiết).

**Database**
- Đảm bảo data được ghi vào database một cách đúng đắn trong quá trình xử lý requests.
- Kiểm tra quá trình migration.
- Kiểm tra việc insert dữ liệu không bình thường thì có được xử lý chính xác hay không.

## Note
1. Fake các data dùng [`Factory`](https://laravel.com/docs/8.x/database-testing#defining-model-factories)

2. Tất cả các test `phải` kiểm tra status code
```dotenv
$response = $this->get(route('admin.advertisement.create'));
$response->assertStatus(Response::HTTP_OK)
```

3. Các dữ liệu ghi vào data base phải kiếm tra tính đúng đắn của nó
```dotenv
- Create, Update
$this->assertDatabaseHas('table_name', data_compare_array);
- Delete
$this->assertDatabaseMissing('table_name', data_compare_array); // có thể dùng ['id' => id_record_deleted]
```

4. Method GET 
- Dữ liệu trả về là HTML, cần kiểm tra trả về file view nào và các text quan trọng
```dotenv
    $response->assertStatus(Response::HTTP_OK)
        ->assertViewIs('backend.advertisement.create')
        ->assertSee('ID')
        ->assertSee('名称')
        ->assertSee('略称')
        ->assertSee('店舗')
        ->assertSee('状態')
        ->assertSee('登録する');
```
- Dữ liệu trả về là JSON
```dotenv
    $response->assertJson(
        fn (AssertableJson $json) => $json->has('data')
        ->has('message')
        ->has('status')
        ->has(
            'data',
            fn ($json) => $json->where('name', $data['name'])
            ->where('name', $data['name'])
            ->where('date_recruitment_end', date('d-m-Y', strtotime($data['date_recruitment_end'])))
            ->where('status', config('data.recruitment_plan_status.waiting'))
            ->where('demand_originator_name', $this->user->name)
            ->has('details.0', fn ($json) => $json->where('estimate_quantity', $data['details'][0]['estimate_quantity'])->etc())
            ->etc()
        )
    );
```

5. Cần kiểm tra các key - value trong session (nếu có)
 - Lưu các key - value vào session
```dotenv
    $response->assertSessionHas('flash_danger', __('You do not have access to do that.'));
```
- Validate error 
```dotenv
    $response->assertSessionHasErrors([
        'id' => '必須項目です。値を必ずご入力してください。',
        'password' => '必須項目です。値を必ずご入力してください。'
    ]);
```

6. Đối với các chức năng sau khi submit form, cần kiểm tra redirect đến url nào
```dotenv
    $response->assertSessionHasErrors('name')
        ->assertRedirect(route('admin.advertisement.create'));
```

## Example
1. Create (Role)
```dotenv
    /** @test */
    public function an_admin_can_access_the_create_role_page()
    {
        $this->loginAsAdmin();
    
        $this->get('/admin/auth/role/create')
            ->assertViewIS('backend.roles.create')
            ->assertSee('Role')
            ->assertOk();
    }
    
    /** @test */
    public function create_role_requires_validation()
    {
        $this->loginAsAdmin();
    
        $response = $this->post('/admin/auth/role');
    
        $response->assertSessionHasErrors('type', 'name');
    }
    
    /** @test */
    public function the_name_must_be_unique()
    {
        $this->loginAsAdmin();
    
        $response = $this->post('/admin/auth/role', ['name' => config('base.access.role.admin')]);
    
        $response->assertSessionHasErrors('name');
    }
    
    /** @test */
    public function a_role_can_be_created()
    {
        Event::fake();
    
        $this->loginAsAdmin();
    
        $this->post('/admin/auth/role', [
            'type' => User::TYPE_ADMIN,
            'name' => 'new role',
            'permissions' => [
                Permission::whereName('admin.access.user.list')->first()->id,
            ],
        ]);
    
        $this->assertDatabaseHas('roles', [
            'type' => User::TYPE_ADMIN,
            'name' => 'new role',
        ]);
    
        $this->assertDatabaseHas('role_has_permissions', [
            'permission_id' => Permission::whereName('admin.access.user.list')->first()->id,
            'role_id' => Role::whereName('new role')->first()->id,
        ]);
    
        Event::assertDispatched(RoleCreated::class);
    }
    
    /** @test */
    public function only_admin_can_create_roles()
    {
        $this->actingAs(User::factory()->admin()->create());
    
        $response = $this->get('/admin/auth/role/create');
    
        $response->assertSessionHas('flash_danger', __('You do not have access to do that.'));
    }
```

2. Edit (Role)
```dotenv
    /** @test */
    public function the_name_is_required()
    {
        $role = Role::factory()->create();
    
        $this->loginAsAdmin();
    
        $response = $this->patch("/admin/auth/role/{$role->id}");
    
        $response->assertSessionHasErrors('name');
    }
    
    /** @test */
    public function a_role_name_can_be_updated()
    {
        Event::fake();
    
        $role = Role::factory()->create();
    
        $this->loginAsAdmin();
    
        $this->patch("/admin/auth/role/{$role->id}", [
            'type' => User::TYPE_ADMIN,
            'name' => 'new name',
            'permissions' => [
                Permission::whereName('admin.access.user.list')->first()->id,
            ],
        ]);
    
        $this->assertDatabaseHas('roles', [
            'type' => User::TYPE_ADMIN,
            'name' => 'new name',
        ]);
    
        $this->assertDatabaseHas('role_has_permissions', [
            'permission_id' => Permission::whereName('admin.access.user.list')->first()->id,
            'role_id' => Role::whereName('new name')->first()->id,
        ]);
    
        Event::assertDispatched(RoleUpdated::class);
    }
    
    /** @test */
    public function the_admin_role_can_not_be_updated()
    {
        $this->loginAsAdmin();
    
        $role = Role::whereName(config('base.access.role.admin'))->first();
    
        $response = $this->patch("/admin/auth/role/{$role->id}", [
            'name' => 'new name',
        ]);
    
        $response->assertSessionHas(['flash_danger' => __('You can not edit the Administrator role.')]);
    
        $this->assertDatabaseMissing('roles', [
            'name' => 'new name',
        ]);
    }
    
    /** @test */
    public function only_admin_can_edit_roles()
    {
        $this->loginAsAdmin();
    
        $role = Role::factory()->create(['name' => 'current name']);
    
        $this->get("/admin/auth/role/{$role->id}/edit")->assertOk();
    }
    
    /** @test */
    public function the_admin_role_can_not_be_edited()
    {
        $this->loginAsAdmin();
    
        $role = Role::whereName(config('base.access.role.admin'))->first();
    
        $response = $this->get("/admin/auth/role/{$role->id}/edit");
    
        $response->assertSessionHas(['flash_danger' => __('You can not edit the Administrator role.')]);
    }
    
    /** @test */
    public function a_non_admin_can_not_edit_roles()
    {
        $this->actingAs(User::factory()->create());
    
        $role = Role::factory()->create(['name' => 'current name']);
    
        $response = $this->get("/admin/auth/role/{$role->id}/edit");
    
        $response->assertSessionHas('flash_danger', __('You do not have access to do that.'));
    }
    
    /** @test */
    public function only_admin_can_update_roles()
    {
        $this->actingAs(User::factory()->admin()->create());
    
        $role = Role::factory()->create(['name' => 'current name']);
    
        $response = $this->patch("/admin/auth/role/{$role->id}", [
            'name' => 'new name',
        ]);
    
        $response->assertSessionHas('flash_danger', __('You do not have access to do that.'));
    
        $this->assertDatabaseHas(config('permission.table_names.roles'), [
            'id' => $role->id,
            'name' => 'current name',
        ]);
    }
```

3. Delete (Role)
```dotenv
    /** @test */
    public function a_role_can_be_deleted()
    {
        Event::fake();
    
        $role = Role::factory()->create();
    
        $this->loginAsAdmin();
    
        $this->assertDatabaseHas(config('permission.table_names.roles'), ['id' => $role->id]);
    
        $this->delete("/admin/auth/role/{$role->id}");
    
        $this->assertDatabaseMissing(config('permission.table_names.roles'), ['id' => $role->id]);
    
        Event::assertDispatched(RoleDeleted::class);
    }
    
    /** @test */
    public function the_admin_role_can_not_be_deleted()
    {
        $this->loginAsAdmin();
    
        $role = Role::whereName(config('base.access.role.admin'))->first();
    
        $response = $this->delete('/admin/auth/role/'.$role->id);
    
        $response->assertSessionHas(['flash_danger' => __('You can not delete the Administrator role.')]);
    
        $this->assertDatabaseHas(config('permission.table_names.roles'), ['id' => $role->id]);
    }
    
    /** @test */
    public function a_role_with_assigned_users_cant_be_deleted()
    {
        $this->loginAsAdmin();
    
        $role = Role::factory()->create();
        $user = User::factory()->create();
        $user->assignRole($role);
    
        $response = $this->delete('/admin/auth/role/'.$role->id);
    
        $response->assertSessionHas(['flash_danger' => __('You can not delete a role with associated users.')]);
    
        $this->assertDatabaseHas(config('permission.table_names.roles'), ['id' => $role->id]);
    }
    
    /** @test */
    public function only_admin_can_delete_roles()
    {
        $this->actingAs(User::factory()->admin()->create());
    
        $role = Role::factory()->create();
    
        $response = $this->delete('/admin/auth/role/'.$role->id);
    
        $response->assertSessionHas('flash_danger', __('You do not have access to do that.'));
    
        $this->assertDatabaseHas(config('permission.table_names.roles'), ['id' => $role->id]);
    }
```
4. List (Role)
```dotenv
    /** @test */
    public function an_admin_can_access_the_role_index_page()
    {
        $this->loginAsAdmin();
    
        $this->get('/admin/auth/role')->assertOk();
    }
    
    /** @test */
    public function only_admin_can_view_roles()
    {
        $this->actingAs(User::factory()->admin()->create());
    
        $response = $this->get('/admin/auth/role');
    
        $response->assertSessionHas('flash_danger', __('You do not have access to do that.'));
    }
```