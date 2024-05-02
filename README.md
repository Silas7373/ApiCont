using HTLAnmeldung.Data;
using HTLAnmeldung.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using System.Text.Json;

namespace HTLAnmeldung.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class AnmeldungController : ControllerBase
    {
        private readonly ApplicationDbContext _context;

        public AnmeldungController(ApplicationDbContext context)
        {
            _context = context;
        }

        [HttpGet("Seed")]
        public IActionResult Seed()
        {
            
            if (_context.departments.Any())
            {
               
                _context.departments.RemoveRange(_context.departments);
                _context.SaveChanges();
            }

            
            var departments = new List<Department>
                {
                    new Department { Id = 1, Name = "INF", Longname = "Die tolle Informatik-Abteilung" },
                    new Department { Id = 2, Name = "ET", Longname = "Elektrotechnik ist immer cool" },
                    new Department { Id = 3, Name = "BT", Longname = "Bautechnik geht auch" }
                };
            _context.departments.AddRange(departments);
            _context.SaveChanges();

            var btRegistrations = JsonSerializer.Deserialize<List<Registration>>(System.IO.File.ReadAllText("JsonTestDateien/BT-Registrations.json"));
            var etRegistrations = JsonSerializer.Deserialize<List<Registration>>(System.IO.File.ReadAllText("JsonTestDateien/ET-Registrations.json"));
            var infRegistrations = JsonSerializer.Deserialize<List<Registration>>(System.IO.File.ReadAllText("JsonTestDateien/INF-Registrations.json"));

            _context.departments.Find(1).Registrations = infRegistrations;
            _context.departments.Find(2).Registrations = etRegistrations;
            _context.departments.Find(3).Registrations = btRegistrations;

            _context.SaveChanges();

            return Ok();
        }


        [HttpPost("GetAllFromDepartments")]
        public IActionResult GetAllFromDepartments([FromBody] List<string> departmentNames)
        {
            var departments = _context.departments
                .Where(d => departmentNames.Contains(d.Name))
                .Include(d => d.Registrations)
                .ToList();

            return Ok(departments);
        }
    }

}
