## Filter & Search Data Using @Query()


#### `employee-bd.service.ts`
```bash
# employee-bd.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Employee } from './employees.entity';
import { ILike, Like, Repository } from 'typeorm';

@Injectable()
export class EmployeeBdService {
    constructor(
        @InjectRepository(Employee) private employeeRepository: Repository<Employee>,
    ) {}

    async create(employeeData: Partial<Employee>): Promise<Employee> {
        const employee = this.employeeRepository.create(employeeData);
        return this.employeeRepository.save(employee);
    }

    async findAll(): Promise<Employee[]> {
        return this.employeeRepository.find();
    }

    async findOne(id: number): Promise<Employee> {
        const employee = await this.employeeRepository.findOneBy({ id });
        if (!employee) {
            throw new NotFoundException(`Employee with ID ${id} not found`);
        }
        return employee;
    }

    async findByName(name: string): Promise<Employee[]> {
        const employee = await this.employeeRepository.find({
            where: { name },
        });
        if (!employee.length) {
            throw new NotFoundException(`Employee with name ${name} not found`);
        }
        return employee;
    }

    // Like is case-sensitive, ILike is case-insensitive
    async searchByName(keyword: string): Promise<Employee[]> {
        return this.employeeRepository.find({
            where: {
                //name: Like(`%${keyword}%`)
                name: ILike(`%${keyword}%`)
            }
        });
    }

    async update(id: number, updateData: Partial<Employee>): Promise<Employee> {
        const employee = await this.employeeRepository.findOneBy({ id });
        if (!employee) {
            throw new NotFoundException(`Employee with ID ${id} not found`);
        }
        const updatedEmployee = Object.assign(employee,  updateData);
        return this.employeeRepository.save(updatedEmployee);
    }

    async delete(id: number): Promise<{ message: string }> {
        const result = await this.employeeRepository.delete({ id });
        if (result.affected === 0) {
            throw new NotFoundException(`Employee with ID ${id} not found`);
        }
        return { message: `Employee with ID ${id} deleted successfully` };
    }

    async search(filters: { name?: string; department?: string }): Promise<Employee[]> {
        const query = this.employeeRepository.createQueryBuilder('employee');

        if (filters.name) {
            query.andWhere('employee.name ILIKE :name', { name: `%${filters.name}%`});
        }
        if (filters.department){
            query.andWhere(`employee.department ILIKE :department`, { department: `%${filters.department}%`});
        }
        return query.getMany();
    }

}
```
---

>#### Note: Get method er modhe id er agei filter dite hobe na hole filter kaj korbe na. code ta dekhle clear hoye jabe

#### `employee-bd.controller.ts`
```bash
import { Body, Controller, Delete, Get, Param, Post, Put, Query } from '@nestjs/common';
import { EmployeeBdService } from './employee-bd.service';
import { Employee } from './employees.entity';

@Controller('employee-bd')
export class EmployeeBdController {
    constructor(private readonly employeeBdService: EmployeeBdService) {}

    @Post()
    async createEmployee(@Body() employeeData: Partial<Employee>) {
        return this.employeeBdService.create(employeeData);
    }

    @Get()
    async findAllEmployees(): Promise<Employee[]> {
        return this.employeeBdService.findAll();
    }

    @Get('filter')
    async filterEmployees(
        @Query('name') name?: string,
        @Query('department') department?: string,
    ): Promise<Employee[]> {
        return this.employeeBdService.search({ name, department });
    }

    // find by id
    @Get(':id')
    async findEmployeeById(@Param('id') id: number): Promise<Employee> {
        return this.employeeBdService.findOne(id);
    }

    // find by name
    @Get('name/:name')
    async findEmployeeByName(@Param('name') name: string): Promise<Employee[]> {
        return this.employeeBdService.findByName(name);
    }

    // search by name (partial match)
    @Get(`search/:keyword`)
    async searchEmployeeByName(@Param('keyword') keyword: string): Promise<Employee[]> {
        return this.employeeBdService.searchByName(keyword);
    }

    @Put(':id')
    async updateEmployee(
        @Param('id') id: number, 
        @Body() updateData: Partial<Employee>): Promise<Employee> {
        return this.employeeBdService.update(id, updateData);
    }

    @Delete(':id')
    async deleteEmployee(@Param('id') id: number): Promise<{ message: string }> {
        return this.employeeBdService.delete(id);
    }

}
```
---

>## OUTPUT
>
><img width="713" height="334" alt="image" src="https://github.com/user-attachments/assets/dc8b38ff-d2e3-4c11-9a9f-39fa948297df" />
---
